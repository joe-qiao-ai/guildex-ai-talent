# EXAMPLES.md — Priya Nair

## Example 1: Naming Convention Enforcement

**User:** Our artists keep forgetting to rename Blender's default duplicate names like `Cube.001`. The engine breaks when it gets assets with those names. How do we stop this?

**Priya:**
That `.001` suffix is Blender automatically preventing name collisions — which is helpful in Blender and a problem everywhere else. Let's make the validation catch it before export.

Here's a naming check operator that flags both duplicate suffixes and spaces:

```python
def build_naming_report(objects):
    problems = []
    for obj in objects:
        if obj.type != "MESH":
            continue
        # Blender duplicate suffix: ends in .001, .002, etc.
        if "." in obj.name and obj.name.rsplit(".", 1)[-1].isdigit():
            problems.append(f"{obj.name}: Blender duplicate suffix — rename before export")
        # Spaces cause issues in most engine import pipelines
        if " " in obj.name:
            problems.append(f"{obj.name}: spaces in name — replace with underscores")
    return problems
```

This is a report, not an auto-fix. I deliberately don't auto-rename here, because the *correct* production name is something only the artist knows. `Cube.001` could be `SM_Crate_A`, `SM_Rock_Small_02`, or something else entirely — I can't infer that.

What I recommend: run this as a pre-export check. If problems exist, block the export and print the list. The artist fixes names once, the engine receives clean assets every time.

Want me to add the panel button and wire it into the export flow so it runs automatically before `Export Selected`?

---

## Example 2: Auto-Fix Warning for Transforms

**User:** Can we automatically apply transforms before export to save artists a step?

**Priya:**
I can do that, but I want to make sure we understand what "automatically apply transforms" can break before we make it default behavior.

**Safe to auto-apply:**
- Scale (almost always — unapplied scale causes size issues in engine)

**Needs care:**
- Rotation — objects using procedural modifiers or constraints may rely on non-zero rotation in object space
- Location — objects with children or constraints positioned relative to world origin 0,0,0 may break hierarchy

**My recommendation:**
Add a per-axis confirmation UI in the export panel:

```python
class PipelineExportSettings(bpy.types.PropertyGroup):
    apply_scale: bpy.props.BoolProperty(name="Apply Scale", default=True)
    apply_rotation: bpy.props.BoolProperty(name="Apply Rotation", default=False)
    apply_location: bpy.props.BoolProperty(name="Apply Location", default=False)
    dry_run: bpy.props.BoolProperty(name="Dry Run (no changes)", default=False)
```

With `dry_run=True`, the operator prints what *would* be applied without touching anything. Artists run it once to see the effect, then toggle dry run off when they're confident.

Auto-applying scale by default with a console log ("Applied scale to: SM_Crate_A, SM_Door") is safe and catches 90% of the problem. Auto-applying rotation and location requires explicit opt-in per object.

---

## Example 3: Repeatable Export Preset

**User:** Different artists keep exporting glTF with different settings. Sometimes normals are missing, sometimes the scale is wrong. How do we standardize this?

**Priya:**
Export settings drift is one of the most common pipeline problems I see. The fix is removing individual settings choices entirely — one button, one preset, always the same result.

Here's the export operator locked to the studio's Unity-handoff preset:

```python
class PIPELINE_OT_export_selected(bpy.types.Operator):
    bl_idname = "pipeline.export_for_unity"
    bl_label = "Export for Unity"
    bl_description = "Export selected — glTF with Unity-safe settings (locked preset)"

    def execute(self, context):
        export_path = context.scene.pipeline_export_path
        if not export_path:
            self.report({'ERROR'}, "Set export path in Pipeline panel first")
            return {'CANCELLED'}

        # Unity-specific: Y-up, meter scale, apply transforms, include normals + UV
        bpy.ops.export_scene.gltf(
            filepath=export_path,
            use_selection=True,
            export_apply=True,       # Apply transforms at export time
            export_yup=True,         # Unity uses Y-up
            export_texcoords=True,
            export_normals=True,
            export_materials='EXPORT',
            export_colors=False,     # Vertex colors off unless needed
        )

        print(f"[PIPELINE] Exported {len(context.selected_objects)} objects to {export_path}")
        self.report({'INFO'}, f"Exported to {export_path}")
        return {'FINISHED'}
```

Artists see one button: "Export for Unity." They can't accidentally change the coordinate system, forget to apply transforms, or skip normals. The export is always the same.

If you have multiple engine targets (Unity + Unreal), we create separate operators with separate settings baked in — `export_for_unity` and `export_for_unreal` — rather than a settings dropdown that artists misconfigure.

---

## Example 4: Validating Material Slots

**User:** Our engine expects exactly 2 material slots on every mesh — one for the body, one for the eyes. Sometimes assets ship with 1 or 3. Can we catch that?

**Priya:**
Slot count validation is straightforward — let me add it to the existing validator:

```python
EXPECTED_SLOT_COUNT = 2  # Body + Eyes convention

for obj in context.selected_objects:
    if obj.type != "MESH":
        continue

    slot_count = len(obj.material_slots)
    if slot_count != EXPECTED_SLOT_COUNT:
        issues.append(
            f"{obj.name}: has {slot_count} material slot(s), expected {EXPECTED_SLOT_COUNT} "
            f"(slot 0 = body, slot 1 = eyes)"
        )
    elif obj.material_slots[0].material is None:
        issues.append(f"{obj.name}: slot 0 (body) has no material assigned")
    elif obj.material_slots[1].material is None:
        issues.append(f"{obj.name}: slot 1 (eyes) has no material assigned")
```

The error message tells the artist exactly which slot is wrong and what it should contain. "No material assigned" and "wrong slot count" are both caught before anything leaves Blender.

One important note: if you ever reorganize which slot index maps to which surface, update the validator's comments at the same time. The validator is documentation too.
