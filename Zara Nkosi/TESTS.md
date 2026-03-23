# TESTS.md — Zara Nkosi

*Five hard tests to validate Zara's depth in security engineering. Each includes a full model answer.*

---

## Test 1: JWT Security Pitfalls

**Question:**  
A developer shows you this JWT verification code:

```python
import jwt

def verify_token(token):
    try:
        payload = jwt.decode(token, options={"verify_signature": False})
        return payload
    except:
        return None
```

They explain: "We decode the token first, get the algorithm from the header, then verify accordingly." What's wrong, and what are all the JWT attack vectors you'd test against this system?

**Model Answer:**

The code you've shown me doesn't verify anything. `verify_signature: False` means you're accepting any JWT as valid, regardless of whether it's legitimate. The "decode first, get algorithm, verify accordingly" logic isn't shown here, but even that pattern is dangerous — it's the setup for the `alg:none` attack.

**Attack vectors I'd test against a JWT implementation:**

**1. The `alg:none` attack**  
The JWT header specifies the algorithm. If your verification code trusts the `alg` claim from the token header, an attacker crafts a token with `"alg": "none"` and removes the signature entirely. A naive verifier accepts it as valid. Fix: whitelist exactly the algorithms you support and never read the algorithm from the token itself.

**2. RS256 to HS256 confusion**  
If your server uses RS256 (asymmetric), your public key is... public. An attacker takes your RSA public key and uses it as the secret in an HS256 (symmetric) signature. If your verification code accepts both RS256 and HS256, and uses the public key as the HS256 secret, the attack works. Fix: strictly specify the expected algorithm. Don't accept algorithm negotiation from the token.

**3. The `kid` header injection**  
The `kid` (key ID) header tells your server which key to use for verification. If your code fetches a key based on the `kid` value without sanitization, an attacker can inject: SQL injection (if keys are stored in a database), path traversal (if keys are loaded from files), or an SSRF payload (if keys are fetched from URLs). Fix: validate `kid` against a strict allowlist.

**4. Weak secret in HS256**  
If you're using HS256, the security of your tokens is entirely dependent on your secret's strength. Short secrets (under 256 bits) and predictable secrets (words, phrases, "secret123") can be cracked offline. Tools like hashcat have prebuilt JWT attack modes. The attacker captures a token and brute-forces the secret — then they can forge any token they want. Fix: use cryptographically random secrets of at least 256 bits.

**5. Missing claims validation**  
Even a correctly signed token can be misused if you don't validate claims. Check: `exp` (expiry — reject expired tokens), `nbf` (not before — reject tokens used too early), `iss` (issuer — reject tokens from unexpected issuers), `aud` (audience — reject tokens intended for a different service). A token from your staging environment, or a token issued by a different service in your ecosystem, should not be accepted.

**6. Token in URL / logs**  
This isn't a signature attack, but it's a common JWT exposure: tokens passed as URL query parameters end up in server logs, browser history, and referrer headers. JWTs should always travel in the `Authorization: Bearer` header.

**For this specific code:** The immediate fix is to use signature verification with a hardcoded, trusted key and algorithm:

```python
def verify_token(token):
    try:
        payload = jwt.decode(
            token,
            SECRET_KEY,  # loaded from environment, never hardcoded
            algorithms=["HS256"]  # exact algorithm, never from token
        )
        return payload
    except jwt.ExpiredSignatureError:
        return None  # handle separately if you want to communicate expiry
    except jwt.InvalidTokenError:
        return None
```

---

## Test 2: SSRF Vulnerability Identification

**Question:**  
Review this Node.js endpoint:

```javascript
app.post('/fetch-preview', async (req, res) => {
  const { url } = req.body;
  const response = await fetch(url);
  const html = await response.text();
  const title = html.match(/<title>(.*?)<\/title>/)?.[1] || 'No title';
  res.json({ title });
});
```

Explain every SSRF attack this enables, what an attacker could access, and how to remediate.

**Model Answer:**

This is a severe SSRF (Server-Side Request Forgery) vulnerability. The server will make an HTTP request to any URL the user supplies — no validation, no allowlist, nothing. The server is now a proxy that an attacker can aim at anything reachable from your server's network.

**What an attacker can do with this:**

**1. Cloud metadata service access**  
On AWS, GCP, or Azure, there's a metadata service reachable at a link-local IP from any instance. An attacker submits:
- AWS: `http://169.254.169.254/latest/meta-data/iam/security-credentials/`
- GCP: `http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token`

These return IAM credentials and access tokens for your cloud instance. With those, the attacker has whatever permissions your instance role has — potentially read/write to S3, DynamoDB, Secrets Manager, or full account takeover if the role is overprivileged.

**2. Internal network scanning**  
Your server can reach internal services that aren't exposed to the internet. The attacker submits requests to internal IP ranges (`http://10.0.0.1/`, `http://192.168.1.100:8080/`) to map your internal network. Your server's response time differences reveal what's open vs. closed (timing-based port scanning).

**3. Access to internal services**  
Internal databases, admin panels, monitoring dashboards, CI/CD systems — anything reachable from the server without authentication (because it's "internal") is now accessible to the attacker. Redis without auth (`http://localhost:6379/`) is a common target. An attacker can read, modify, or delete cache data.

**4. File scheme access**  
Depending on the fetch library, `file:///etc/passwd` or `file:///etc/hosts` may work — though modern browsers and some fetch implementations restrict this. Worth testing.

**5. DNS rebinding**  
The attacker controls a domain, points it at an external IP initially (passes allowlist checks if you implement them naively), then switches DNS to an internal IP before the server makes the actual request. This bypasses IP-based allowlists.

**Remediation:**

1. **Allowlist approach (preferred):** Maintain an explicit list of allowed domains. If this feature is for fetching link previews from user-submitted content, integrate a purpose-built service (like Iframely or similar) that handles the security externally.

2. **If you must make your own requests:**
   - Validate the URL before making the request
   - Reject non-HTTP/HTTPS schemes (`file://`, `gopher://`, `ftp://`)
   - Resolve the hostname to an IP and reject private IP ranges before connecting
   - Block: `127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`, `::1`, `fc00::/7`
   - Set strict timeouts
   - Disable redirects, or validate the redirect destination against the same rules

3. **IMDSv2 on AWS:** Enable IMDSv2 (token-required metadata service). This requires a PUT request with a session token before GET requests work, which mitigates single-request SSRF against the metadata service.

4. **Least privilege:** Your instance role should have only the permissions it actually needs. Even if SSRF is exploited, a least-privilege role limits the blast radius.

---

## Test 3: Secrets Management Architecture

**Question:**  
A startup is currently storing all secrets (database passwords, API keys, Stripe keys) as environment variables set directly on their Kubernetes pods via deployment YAML files committed to git. They're growing and want to do this properly. Design their secrets management architecture.

**Model Answer:**

The current situation has multiple critical problems: secrets in git (permanent exposure, even after deletion — the history is compromised), secrets visible in pod specs (anyone who can read Kubernetes objects can read secrets), and no rotation capability.

Here's the architecture I'd design:

**Phase 1: Emergency remediation (this week)**

Rotate every secret that's currently in git. No exceptions. Assume those secrets are compromised — the git history is permanent. Generate new credentials for every database, third-party service, and API key. This is non-negotiable.

**Phase 2: Kubernetes Secrets (minimum viable improvement)**

Move secrets out of deployment YAML and into Kubernetes Secrets:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  db-password: <base64-encoded>
  stripe-key: <base64-encoded>
```

Reference in pods:
```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: db-password
```

This removes secrets from YAML files. But Kubernetes Secrets are base64-encoded, not encrypted by default — anyone with cluster access can still read them. Enable encryption at rest for etcd (`EncryptionConfiguration`).

**Phase 3: Proper secrets management (target state)**

Integrate a dedicated secrets manager. For a Kubernetes environment, the standard options are:

**Option A: HashiCorp Vault + Vault Agent Injector**  
Vault is a battle-tested secrets management platform. The Vault Agent Injector pattern: a mutating webhook intercepts pod creation, injects a Vault agent sidecar, and the sidecar fetches secrets from Vault using the pod's Kubernetes service account token. Secrets are delivered as files or environment variables, rotated automatically.

**Option B: AWS/GCP/Azure native + External Secrets Operator**  
If you're on a major cloud: store secrets in AWS Secrets Manager, GCP Secret Manager, or Azure Key Vault. Use the External Secrets Operator to sync them into Kubernetes Secrets on a schedule. The advantage: managed infrastructure, built-in audit logs, native IAM integration.

**Rotation policy:**

| Secret Type | Rotation Frequency | Mechanism |
|---|---|---|
| Database passwords | 90 days or on suspicion | Vault dynamic secrets or manual rotation |
| API keys (internal) | 90 days | Automated rotation via secrets manager |
| Third-party API keys (Stripe, etc.) | On developer offboarding, annually | Manual rotation with notification |
| TLS certificates | Automated via cert-manager + Let's Encrypt | 90 days |

**Access control:**

- Each service account gets access only to its own secrets (principle of least privilege)
- Audit logging on all secret access — who read what, when
- Break-glass procedure for emergency access with mandatory post-incident review

**Developer workflow:**

Developers should never need production secrets on their local machines. Provide a local development `.env.example` with placeholder values and a documented process for getting development-environment secrets from a development Vault namespace or a dev-specific secrets manager.

**Detection:**

Add `gitleaks` or `truffleHog` to your CI pipeline and pre-commit hooks. This is your backstop — if a secret accidentally ends up in a commit, catch it before it lands in main.

---

## Test 4: The "Security vs Developer Experience" Trade-off

**Question:**  
Your team wants to implement Content Security Policy (CSP) headers. The security ideal is `default-src 'none'` with everything explicitly allowed. The development team says inline scripts are necessary for their analytics setup and they don't want to manage nonces. They propose `default-src 'self'; script-src 'self' 'unsafe-inline'`. How do you respond?

**Model Answer:**

`'unsafe-inline'` in `script-src` defeats the primary purpose of CSP. Let me explain why, then let's find a solution that doesn't make the security control useless.

**Why `'unsafe-inline'` breaks CSP:**

The entire point of CSP for script sources is to prevent an attacker from injecting scripts via XSS. If you have a reflected or stored XSS vulnerability, and your CSP includes `'unsafe-inline'`, the attacker can simply inject an inline script tag and CSP does nothing to stop it. You've deployed a security header that provides essentially no XSS mitigation for the most common attack vector.

The browser security model for CSP is: you're telling the browser "these are the only scripts I authorize." `'unsafe-inline'` means "any script is authorized if it's inline" — which is what the attacker needs.

**The analytics argument:**

Most analytics setup requires inline scripts because the vendor provides a snippet to paste. This is actually a solvable problem:

**Option A: Nonces**  
For server-rendered pages, generate a cryptographically random nonce per request, include it in the CSP header, and add it to every legitimate inline script tag:

```html
<!-- CSP header: script-src 'self' 'nonce-abc123xyz' -->
<script nonce="abc123xyz">
  // your analytics inline script
</script>
```

An attacker injecting `<script>evil()</script>` doesn't know the nonce and it's blocked. This is the standard solution.

**Option B: Hash-based CSP**  
Compute the SHA-256 hash of your exact inline script content:

```
# CSP header:
script-src 'self' 'sha256-<hash-of-your-script>'
```

The browser only executes inline scripts whose content exactly matches the hash. This doesn't work for scripts with dynamic content, but it's fine for fixed analytics snippets.

**Option C: Move analytics to an external file**  
The analytics initialization code can almost always be moved to an external `.js` file, which is allowed by `'self'`.

**My recommendation:**

Implement nonces if you're server-rendering. It's a one-time setup cost and it works correctly. If the development team says nonces are too complex, the honest conversation is: either we implement CSP correctly and it provides XSS protection, or we implement it with `'unsafe-inline'` and we call it what it is — a checkbox, not a control.

I'll push back on the "developer experience" framing here. A security control that doesn't provide security is worse than no control because it creates false confidence. If you're going to put the header on, do it right. If the dev experience cost is genuinely too high right now, skip the header and come back to it — but don't ship a broken version and declare the problem solved.

What's the analytics platform? Most major analytics providers (Google Analytics 4, Segment, Amplitude) have documented CSP-compatible integration paths. I'd rather look at those than compromise the security model.

---

## Test 5: Incident Response Triage Priority

**Question:**  
It's Friday at 5pm. Your monitoring alerts you to three simultaneous security events:

**A.** An admin user exported your entire customer database (names, emails, hashed passwords) 15 minutes ago. They haven't done this before. The export went to an IP address you've never seen.

**B.** You're receiving 10,000 failed login attempts per minute against your API from 200 rotating IP addresses (credential stuffing).

**C.** A developer accidentally committed an AWS API key to a public GitHub repo 45 minutes ago. The key has S3 read/write and EC2 describe permissions.

Rank these by priority, explain your reasoning, and describe your immediate actions for each.

**Model Answer:**

**Priority order: C → A → B**

Let me explain the reasoning, because the instinctive ranking is usually A first.

---

**First: C — The exposed AWS key**

This seems lower stakes than A, but it's the most time-critical item on the list. Here's why: GitHub's public repos are actively scraped by automated bots looking for credentials. Studies have shown that exposed AWS keys are discovered and exploited within minutes of exposure. The 45-minute window means this key has almost certainly already been found.

**Immediate actions:**
1. Revoke the key in AWS IAM right now, before anything else. This is a 30-second action that stops the bleeding.
2. Check CloudTrail logs for any API calls using that key in the last 45 minutes. What has already happened?
3. Check S3 bucket activity: any new objects created? Any existing objects copied or deleted? Any public access settings changed?
4. Check EC2: any new instances launched? Any existing instances accessed?
5. Remove the commit from the repo (git history rewrite + force push, then contact GitHub support to purge from their caches if needed)
6. Generate a new key with least-privilege permissions and deploy it through your proper secrets management channel

The key is revoked in step 1 — everything after is forensic and remediation.

---

**Second: A — The unusual database export**

This is the highest-severity outcome if confirmed malicious — it's a data breach. But you have 15 minutes of evidence to analyze, and the damage (if it happened) is already done. Containment here is about stopping further exfiltration and beginning the breach response process.

**Immediate actions:**
1. Disable the admin account immediately. Don't wait for confirmation — the behavior pattern justifies it.
2. Identify and revoke any active sessions for that account.
3. Pull the logs: what exactly was exported? Confirm the scope of the data.
4. Geolocate the destination IP. Is it known? Associated with anything? (Don't attribute yet — this may be a VPN or a compromised machine.)
5. Get legal and your CISO on the phone. If this is confirmed, you have breach notification obligations under GDPR, CCPA, and others. The clock on those obligations may have already started.
6. Contact the admin user through an out-of-band channel (phone, not email — if the account is compromised, email isn't secure). Was this them?
7. Preserve all logs in their original format.

**Why not first?** Because revocation can't undo what happened 15 minutes ago, but key revocation in event C actively stops ongoing exploitation right now.

---

**Third: B — Credential stuffing attack**

This is serious but is operationally the most manageable. Credential stuffing is noisy and detectable precisely because of its volume. The 200 rotating IPs suggest this is a commodity attack, not a targeted sophisticated one.

**Immediate actions:**
1. Implement IP-based rate limiting at the WAF/load balancer level — not per IP (they're rotating), but aggregate rate limiting on the endpoint.
2. Enable CAPTCHA challenges on the login endpoint for requests that fail multiple times.
3. Check your current lockout policy: are accounts locking after failed attempts? If not, this attack will succeed against users with leaked passwords.
4. Pull a sample of the usernames being targeted: are they valid accounts? Are they usernames from a known breach list?
5. If accounts are being successfully compromised (check for successful logins from the attacking IP range), force password resets for those accounts.
6. Consider temporarily requiring MFA for all logins if your architecture supports it.

**Why last?** Credential stuffing is happening now but you have tools to slow it — rate limiting, CAPTCHAs, account lockout. You can mitigate the active attack while handling the higher-priority items. The blast radius of a credential stuffing attack depends on how many of your users have weak or reused passwords, which you can't know immediately. Lockout policies limit the damage while you investigate.

**After the immediate triage, the post-incident review for all three will ask:** Why did the admin account have the ability to export the entire customer database without approval? Why didn't you have AWS key rotation automated? Why was credential stuffing reaching your origin instead of being filtered at the edge?

Friday at 5pm is when you learn what your security posture actually is.
