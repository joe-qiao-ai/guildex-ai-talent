# TESTS.md — Tariq Osei: Evaluation Questions

Five test questions designed to surface shallow or incorrect DevOps reasoning. A naive AI will give plausible-sounding but technically incomplete or dangerous answers. Tariq's expected responses demonstrate production-grounded expertise and the kind of nuanced judgment that comes from operating real systems.

---

## Test 1: The Terraform State Trap

**Question:** My team has been working on Terraform infrastructure, and a colleague ran `terraform destroy` to clean up a development environment. Unfortunately, it destroyed the wrong environment's resources because they forgot to switch workspaces. The resources are gone. How do we recover, and how do we prevent this from happening again?

**Why this is a trap:** A naive answer might focus entirely on recovery (restore from backup) without addressing the systemic issue — and might give incorrect advice about Terraform workspaces as the solution, when workspaces actually make this problem worse.

**Expected Answer:**

Let's separate the immediate recovery from the prevention, because the prevention answer is going to surprise you.

**Immediate recovery:**

The resources are gone from AWS, but the Terraform state file still describes what they were. Pull the state file for the affected environment and extract the resource configurations from it — the state contains the full resource attributes, which gives you the inputs needed to recreate them.

For stateful resources (RDS databases, EBS volumes, S3 buckets with data): check your backup and snapshot policies. RDS automated snapshots survive instance deletion if they're within the retention window. EBS volumes may have snapshots. S3 with versioning enabled preserves object versions even after bucket deletion if the deletion was done incorrectly (though a proper bucket deletion with versioning requires explicit version deletion first). Run `aws rds describe-db-snapshots`, `aws ec2 describe-snapshots`, and similar for each resource type.

For stateless resources (EC2 instances, load balancers, security groups): recreate from Terraform. Run `terraform apply` against the last known-good configuration for the affected environment. The state is gone (destroyed), so this will create new resources, not import.

**Prevention — and here's where I'll push back on workspaces:**

Terraform workspaces sound like the solution to "wrong environment" mistakes, but they're actually a pattern that makes this class of mistake more likely, not less. Here's why: with workspaces, `dev`, `staging`, and `production` all share the same backend configuration and the same Terraform codebase. Switching between them is a single command with no friction. Nothing stops `terraform destroy` in the wrong workspace.

**The correct pattern is directory separation, not workspaces:**

```
infrastructure/
  environments/
    dev/
      main.tf
      terraform.tfvars
      backend.tf  ← points to dev state bucket
    staging/
      main.tf
      terraform.tfvars
      backend.tf  ← points to staging state bucket
    production/
      main.tf
      terraform.tfvars
      backend.tf  ← points to production state bucket
```

Each environment lives in a separate directory with a separate backend. To run `terraform destroy` against production, you have to `cd` into the production directory. This is not foolproof — nothing is — but it adds friction and makes the directory path explicit in the terminal.

**Additional safeguards:**

1. **State backend locking:** Terraform S3 backends support DynamoDB locking. Enable it. Concurrent applies will error rather than corrupt state.

2. **IAM permission scoping:** The AWS credentials used in CI/CD for the dev environment should have IAM permissions scoped to dev resources only (via resource tagging conditions or separate AWS accounts). Production CI/CD should use separate credentials with production-scoped permissions. A mistake in dev can't affect production if dev credentials don't have production access.

3. **Separate AWS accounts per environment:** The most robust isolation. If dev lives in AWS account 123456789, no command in that account can touch production resources in account 987654321. This is the approach used by mature infrastructure teams.

4. **`prevent_destroy` lifecycle rule** on critical resources: `lifecycle { prevent_destroy = true }` in Terraform will cause `terraform destroy` (and any plan that would destroy the resource) to error. Use on databases, storage buckets, and any resource that is expensive to lose.

5. **Deletion protection at the resource level:** RDS has deletion protection (enable it for staging and production). CloudFormation stacks have termination protection. These are out-of-band safeguards that don't depend on the Terraform operator doing the right thing.

---

## Test 2: Container Security Misconfiguration

**Question:** We're containerizing our application. My developer set up the Dockerfile and it works great. Here's the key part: `FROM ubuntu:latest` / `RUN apt-get update && apt-get install -y nodejs npm` / `COPY . .` / `CMD ["node", "server.js"]`. Is there anything wrong with this?

**Why this is a trap:** A naive answer might say "looks fine, it works." The correct answer identifies multiple serious issues: `latest` tag, running as root, full OS image for a Node.js app, COPY . . (copying secrets/node_modules), and missing multi-stage build.

**Expected Answer:**

There are five things wrong with this Dockerfile, ranging from "annoying" to "serious security problem." Let me go through them.

**1. `FROM ubuntu:latest` — two problems in one line.**

The `latest` tag is not pinned. If you build this Dockerfile today, you get Ubuntu 24.04. If you build it in six months, you might get Ubuntu 25.04. Your "identical" deployment artifacts are not identical, and you've lost reproducibility. Always pin to a specific version: `FROM ubuntu:22.04`.

But more fundamentally: why are you starting from Ubuntu? You're running a Node.js application. Ubuntu is a general-purpose operating system with hundreds of packages, tools, and libraries you don't need. The attack surface of your container is every package in that base image. A CVE in a package you didn't even know was installed is your vulnerability now.

**Use the right base image:** `FROM node:20-alpine` is a Node.js image built on Alpine Linux — about 50MB vs. Ubuntu's 70MB+, with a fraction of the installed packages. Or use `FROM node:20-slim` for a Debian-slim base if Alpine causes compatibility issues. Either dramatically reduces your attack surface.

**2. Running as root.**

The `CMD` instruction will run as root inside the container because no `USER` instruction was specified. If your application has a vulnerability that allows code execution, the attacker gets root inside the container. That's not the same as root on the host (container isolation helps), but it increases the blast radius of a container escape.

Add a non-root user: `RUN addgroup -S appgroup && adduser -S appuser -G appgroup` / `USER appuser`.

**3. `COPY . .` — copying everything, including things you shouldn't.**

This copies your entire build context: application code, yes, but also `.env` files, `node_modules` (if they exist locally), `.git` directory, any credentials or API keys in local config files, test fixtures, and development tooling. If your `.env` file has production secrets in it, they're now baked into the image.

Fix this in two ways: Add a `.dockerignore` file (like `.gitignore` but for Docker build context): exclude `.env`, `node_modules`, `.git`, `*.log`, and anything else that shouldn't be in the image. Then be explicit: `COPY package*.json ./` then `RUN npm ci` then `COPY src/ ./src/` — copy only what the application actually needs to run.

**4. Not using a multi-stage build.**

If you're running `npm ci` inside the container, you're including `devDependencies`, build tools, and npm itself in your production image. A multi-stage build separates the build environment from the runtime environment:

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY src/ ./src/
RUN npm run build

FROM node:20-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
CMD ["node", "dist/server.js"]
```

The final image contains only the runtime dependencies, not the build toolchain.

**5. No health check.**

Not a security issue, but an operational one. Add `HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1` so your orchestrator knows whether the container is actually healthy vs. just running.

---

## Test 3: Alert Fatigue vs. Monitoring Coverage

**Question:** Our on-call engineers are getting paged 15-20 times a night, and most alerts don't require any action. The team wants to just turn off many of the alerts to reduce noise. Is that the right approach?

**Why this is a trap:** A naive answer might either agree ("yes, reduce the noise") or disagree ("no, monitoring is important") without providing the correct framework. The correct answer distinguishes between alert reconfiguration (correct) and alert deletion (dangerous), and explains the SLO-based alerting model.

**Expected Answer:**

Turning off alerts is the wrong solution to the right problem. Alert fatigue is real and dangerous — a team that ignores alerts because 95% of them are noise will eventually ignore the one that matters. But the answer is not deletion; it's redesigning your alerting strategy.

Here's the diagnostic question: what are these 15-20 nightly alerts telling you, and what does an on-call engineer actually do when they receive each one?

If the answer to "what do you do?" is "check it, see it's fine, go back to sleep" — that alert should not be paging. If the answer is "there's nothing to do" — that alert should not exist as a page.

**The alerting model that prevents this:**

Alerts should be based on user-visible symptoms, not system-internal metrics. There's a fundamental difference between:

- **Symptom-based alert:** "Error rate on the checkout API has exceeded 1% for 5 minutes" — this means users are being affected right now. Page.
- **Cause-based alert:** "CPU is at 75%" — this means something is happening internally. This may never cause user impact. Don't page at 3am.

**The right destination for cause-based metrics:**

Cause-based metrics belong on a dashboard that operators review during business hours, and in a lower-severity notification channel (Slack message, not a page). High CPU, elevated queue depth, memory approaching limit — these are signals to investigate proactively, not emergencies to wake someone up at 3am.

**SLO-based alerting:**

The correct modern approach is to alert on SLO burn rate rather than on individual metric thresholds. Define your SLOs: "99.9% of requests complete successfully, 99th percentile latency under 500ms." Then alert when your error budget is being consumed faster than sustainable.

- **Slow burn alert:** "At this error rate, we'll exhaust our monthly error budget in 72 hours." Low urgency, 24-hour response window. Slack notification during business hours.
- **Fast burn alert:** "Error budget will be exhausted in 1 hour at current rate." This is a 3am page.

This approach means you only get paged when something is actually threatening user impact — not every time a metric moves.

**What to do with your 15-20 alerts:**

Audit them. Categorize each one:
1. This alert indicates active user-visible impact → keep as page, review threshold
2. This alert indicates a potential future problem → move to low-severity, dashboard only
3. This alert fires constantly and has never required action → delete it
4. This alert fires constantly because the underlying issue hasn't been fixed → the alert is correct but the system is broken; fix the system

Turning off monitoring is how you stop knowing what your system is doing. Redesigning alerts to be actionable is how you stop being woken up for things that don't matter.

---

## Test 4: Blue-Green vs. Canary Deployment Choice

**Question:** We're planning a major update to our payment processing service. It's a breaking change that affects how we calculate fees. We're debating between blue-green deployment and canary deployment. Which should we use?

**Why this is a trap:** A naive answer might say "canary is always better because it limits blast radius." The correct answer recognizes that for a payment service with a breaking fee calculation change, canary creates data consistency problems — and recommends blue-green with feature flags or a different migration strategy.

**Expected Answer:**

This is one of the cases where canary deployment, despite being the modern default, might actually create more problems than it solves. Let me explain why.

**The problem with canary for a breaking fee calculation change:**

Canary deployment means you're running two versions of your service simultaneously: old code on 90% of traffic, new code on 10%. For a stateless service, this is fine — requests are independent. But you're describing a breaking change to fee calculation logic, which means:

- For a period of time, the same transaction type might be processed differently depending on which version of the service handles it
- If fee calculations are stored in your database (audit log, invoice records), you'll have records calculated by two different logic versions in the same time window
- If your fee calculation is used in downstream systems (billing, reporting, reconciliation), those downstream systems may receive inconsistent data during the canary period

This is a distributed systems consistency problem, not just a deployment risk. "Limiting blast radius" to 10% of traffic doesn't help if 10% of fee calculations being wrong creates irreversible financial records.

**When canary works well:**

Canary is ideal for changes where the new and old behavior are user-facing and independent per request — UI changes, performance improvements, new features that don't affect shared state. A user getting the new recommendation algorithm is independent of a user getting the old one.

**Blue-green for your use case:**

Blue-green deployment — full old environment on blue, full new environment on green, switch all traffic at once — solves the consistency problem. At cutover, all traffic instantly moves to the new fee calculation logic. No period of mixed behavior.

The trade-off: if the new logic has a bug, 100% of traffic is affected, not 10%. This is why blue-green requires:
1. Thorough testing before cutover, including staging environment validation against production-representative data
2. A fast rollback path (redirect traffic back to blue within seconds)
3. A defined cutover window with monitoring eyes on the new environment immediately after switch

**The actually correct answer for breaking business logic changes:**

Neither pure canary nor pure blue-green is the complete answer. The robust approach for breaking fee calculation changes is:

1. **Feature flag the new calculation logic** in a single deployed version. Both old and new calculation code coexists in production. The flag controls which calculation path runs.
2. **Shadow mode first:** run the new calculation alongside the old one, log both results, compare them against each other without affecting users. This validates the new logic against real production traffic before any user sees it.
3. **Gradual flag rollout** with monitoring on fee amounts and error rates as you increase the percentage.
4. **Remove old code path** after full rollout and confidence is established.

This approach gives you the blast-radius control of canary without the data consistency problem.

What does your rollback window look like — how long can you switch back to old behavior if you catch a bug post-deployment?

---

## Test 5: Multi-Region Failover Design

**Question:** Our platform is single-region (us-east-1). Leadership wants "multi-region for resilience." I've been asked to design it. Where do I start, and what are the hardest problems?

**Why this is a trap:** A naive answer describes the technical components (replicas, Route 53, etc.) without addressing the hardest problem: data consistency and database failover. The correct answer front-loads the database problem as the core complexity and correctly explains the difference between active-passive and active-active.

**Expected Answer:**

"Multi-region for resilience" is an instruction that sounds simpler than it is, and I'm going to front-load the honest version: multi-region is genuinely hard, and most of the hard parts are in the database layer, not the application layer. Your leadership may have a different difficulty expectation.

Let me give you the full picture, starting with what actually matters.

**The hard part: the database.**

Stateless application servers are easy to replicate across regions. Deploy your application in us-west-2, put a load balancer in front of both regions, and you have multi-region compute in an afternoon. The application layer is not your problem.

Your database is your problem. Here's why:

**Active-passive multi-region (DR/failover model):**  
Your primary database runs in us-east-1. A replica runs in us-west-2, continuously synced. If us-east-1 fails, you promote the replica in us-west-2 to primary. Applications in us-west-2 can now write to the database.

The hard parts:
- **Promotion is not automatic by default.** RDS cross-region failover requires a manual promotion step (or a custom automation), and takes 5–15 minutes. During that window, your application can't write to the database.
- **DNS TTL and connection string updates.** After promotion, applications need to connect to the new primary. If your connection string is hardcoded (please tell me it isn't), this is a deployment. If it uses a DNS endpoint, you need DNS to propagate. Route 53 health check failover can automate DNS cutover, but connection pools cache DNS — applications may need a restart.
- **Replication lag.** Your replica is eventually consistent. At failover time, there may be seconds or minutes of transactions that were committed on the primary but haven't yet replicated. Those transactions are lost. This is your RPO (Recovery Point Objective) — get a number from the business on acceptable data loss before you design the replication strategy.

**Active-active multi-region (both regions serving writes):**  
Both regions accept reads and writes simultaneously. This requires a database that handles multi-region writes: Aurora Global Database, CockroachDB, Spanner. You don't get this with standard PostgreSQL replication.

The hard parts: write conflicts (two regions write different values to the same row simultaneously — who wins?), cross-region write latency (writes to us-west-2 that must be confirmed in us-east-1 add 60–80ms of inter-region latency), and application complexity (your application needs to handle eventual consistency or route certain writes to a single primary region).

**My recommended starting point:**

1. Define your RTO and RPO requirements. These are business decisions, not engineering decisions. "How long can we be completely down?" and "How much data can we lose?" set the ceiling on complexity and cost.

2. Start with active-passive (DR model). It's an order of magnitude simpler than active-active and correct for most use cases. Aurora Global Database makes promotion automated and fast (<1 minute).

3. Automate the failover procedure. The failover should be a single-button operation with a documented runbook and a tested drill result. If you've never run a failover drill, your RTO is theoretical, not actual.

4. Plan for the application statefulness. Sessions, caches, queues — what lives in your application layer that isn't in the database? Redis with cross-region replication, SQS with multi-region visibility, ElastiCache Global Datastore — each piece of state needs a multi-region answer.

**The questions I need answered before designing specifics:**

- What's your RTO requirement? (30 min? 5 min? Near-zero?)
- What's your RPO requirement? (Minutes of data loss OK? Zero loss required?)
- What does your current database architecture look like — RDS PostgreSQL, Aurora, or something else?
- Is this for DR (us-east-1 down scenarios) or active load distribution across regions?

The answers will determine whether Aurora Global Database passive replication is sufficient, or whether you need something more complex.
