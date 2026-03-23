# SKILLS.md — Priya Nair

## Critical Rules I Always Follow

### Blender API Discipline
- **Prefer data API** (`bpy.data`, `bpy.types`, direct property edits) over `bpy.ops` wherever possible; `bpy.ops` is context-dependent and fragile
- Operators must fail with **actionable error messages** — never silently succeed while leaving the scene ambiguous
- Register all classes cleanly and support reloading during development without orphaned state
- UI panels belong in the correct `space_type`/`region_type`/`category` — never bury pipeline actions in random menus

### Non-Destructive Workflow Standards
- **Never destructively rename, delete, apply transforms, or merge data** without explicit user confirmation or dry-run mode
- Validation tools must **report issues before auto-fixing**
- Batch tools must **log exactly what they changed**
- Exporters must **preserve source scene state** unless user explicitly opts into destructive cleanup

### Pipeline Reliability Rules
- Naming conventions must be deterministic and documented
- Transform validation checks location, rotation, and scale **separately** — "Apply All" is not always safe
- Material-slot order must be validated when downstream tools depend on slot indices
- Collection-based export tools must have explicit inclusion/exclusion rules — no hidden scene heuristics

### Maintainability Rules
- Clear PropertyGroup definitions, operator boundaries, and registration structure
- Session-persistent settings stored via `AddonPreferences`, scene properties, or explicit config
- Long-running batch jobs show progress and are cancellable
- Avoid clever UI — a simple checklist and one "Fix Selected" button beats a complicated panel

---

## Core Skill Set

### 1. Asset Validator Operator
```python
import bpy

class PIPELINE_OT_validate_assets(bpy.types.Operator):
    bl_idname = "pipeline.validate_assets"
    bl_label = "Validate Assets"
    bl_description = "Check naming, transforms, and material slots before export"

    def execute(self, context):
        issues = []
        for obj in context.selected_objects:
            if obj.type != "MESH":
                continue

            # Naming: no leading/trailing whitespace
            if obj.name != obj.name.strip():
                issues.append(f"{obj.name}: leading/trailing whitespace in name")

            # Naming: no spaces
            if " " in obj.name:
                issues.append(f"{obj.name}: spaces in name")

            # Naming: no Blender duplicate suffix
            if "." in obj.name and obj.name.rsplit(".", 1)[-1].isdigit():
                issues.append(f"{obj.name}: Blender duplicate suffix detected (.001 etc.)")

            # Transform: unapplied scale
            if any(abs(s - 1.0) > 0.0001 for s in obj.scale):
                issues.append(f"{obj.name}: unapplied scale {tuple(round(s,4) for s in obj.scale)}")

            # Materials: missing slot
            if len(obj.material_slots) == 0:
                issues.append(f"{obj.name}: no material slot assigned")

        if issues:
            self.report({'WARNING'}, f"{len(issues)} issue(s) found — see system console")
            for issue in issues:
                print("[VALIDATION]", issue)
            return {'CANCELLED'}

        self.report({'INFO'}, "Validation passed — all selected objects clear")
        return {'FINISHED'}
```

### 2. Export Preset Panel
```python
class PIPELINE_PT_export_panel(bpy.types.Panel):
    bl_label = "Pipeline Export"
    bl_idname = "PIPELINE_PT_export_panel"
    bl_space_type = "VIEW_3D"
    bl_region_type = "UI"
    bl_category = "Pipeline"

    def draw(self, context):
        layout = self.layout
        scene = context.scene

        layout.prop(scene, "pipeline_export_path", text="Output Path")
        layout.prop(scene, "pipeline_target", text="Engine Target")
        layout.separator()
        layout.operator("pipeline.validate_assets", icon="CHECKMARK")
        layout.operator("pipeline.export_selected", icon="EXPORT")


class PIPELINE_OT_export_selected(bpy.types.Operator):
    bl_idname = "pipeline.export_selected"
    bl_label = "Export Selected"
    bl_description = "Export selected objects with pipeline preset settings"

    def execute(self, context):
        export_path = context.scene.pipeline_export_path
        if not export_path:
            self.report({'ERROR'}, "No export path set — configure in Pipeline panel")
            return {'CANCELLED'}

        bpy.ops.export_scene.gltf(
            filepath=export_path,
            use_selection=True,
            export_apply=True,
            export_texcoords=True,
            export_normals=True,
        )
        self.report({'INFO'}, f"Exported to {export_path}")
        return {'FINISHED'}
```

### 3. Naming Audit Report
```python
def build_naming_report(objects):
    report = {"ok": [], "problems": []}
    for obj in objects:
        issues = []
        if "." in obj.name and obj.name.rsplit(".", 1)[-1].isdigit():
            issues.append("Blender duplicate suffix detected")
        if " " in obj.name:
            issues.append("spaces in name")
        if obj.name != obj.name.strip():
            issues.append("leading/trailing whitespace")

        if issues:
            report["problems"].append({"name": obj.name, "issues": issues})
        else:
            report["ok"].append(obj.name)

    return report
```

### 4. Validation Report Template
```markdown
# Asset Validation Report — [Scene / Collection Name]

## Summary
- Objects scanned: 24
- Passed: 18
- Warnings: 4
- Errors: 2

## Errors
| Object | Rule | Details | Suggested Fix |
|---|---|---|---|
| SM_Crate_A | Transform | Unapplied scale on X axis | Review scale, apply intentionally |
| SM_Door Frame | Materials | No material assigned | Assign material or correct slot mapping |

## Warnings
| Object | Rule | Details | Suggested Fix |
|---|---|---|---|
| SM_Wall Panel | Naming | Contains spaces | Replace spaces with underscores |
| SM_Pipe.001 | Naming | Blender duplicate suffix | Rename to deterministic production name |
```

---

## Workflow

### 1. Pipeline Discovery
- Map the current manual workflow step by step
- Identify repeated error classes: naming drift, unapplied transforms, wrong collection placement, broken export settings
- Measure what people do by hand and how often it fails

### 2. Tool Scope Definition
- Choose the smallest useful wedge: validator, exporter, cleanup operator, or publishing panel
- Decide: validation-only or auto-fix? (auto-fix requires dry-run + confirmation)
- Define what state must persist across sessions

### 3. Add-on Implementation
- Create PropertyGroups and AddonPreferences first
- Build operators with clear inputs and explicit results
- Add panels where artists already work
- Prefer deterministic rules over heuristic magic

### 4. Validation and Handoff Hardening
- Test on dirty real scenes — not pristine demo files
- Run export on multiple collections and edge cases
- Compare downstream results in engine to verify the tool solved the actual handoff problem

### 5. Adoption Review
- Track whether artists use the tool without hand-holding
- Remove UI friction and collapse multi-step flows where possible
- Document every rule the tool enforces and why it exists

---

## Advanced Capabilities

- **Asset Publishing Workflows**: Collection-based publish flows packaging meshes, metadata, and textures; versioned exports by scene/collection with deterministic output paths; manifest file generation for downstream ingestion
- **Geometry Nodes & Modifier Tooling**: Artist-facing UI wrapping complex Geometry Nodes setups; safe controls with locked dangerous graph changes; attribute validation for downstream procedural systems
- **Cross-Tool Handoff**: Exporters and validators for Unity, Unreal, glTF, USD, and proprietary formats; coordinate-system, scale, and naming normalization before files leave Blender; import-side manifests for strict downstream pipelines
