# Design Rules in the InHouse template

The InHouse template stores its manufacturing defaults in the project settings file, not in a populated PCB rule table.

That means the rule values are present in the project JSON and are used when a new board is created from the template, but the empty template PCB itself has no board objects or editable rule rows to show in the GUI. As a result, the KiCad "Tools → Design Rule Editor" dialog can look empty when you open the blank template.

## Why the values are still there

The actual DRC values live in:

- `sources/templates/InHouse/InHouse.kicad_pro`
- `packages/htl-inhouse/templates/InHouse/InHouse.kicad_pro`

They appear under:

- `board.design_settings.rules`
- `board.design_settings.track_widths`
- `board.design_settings.via_dimensions`
- `net_settings.classes`

These are the settings KiCad uses as the default board constraints for a new project created from this template.

## Why the editor is empty

The blank template PCB is intentionally minimal:

- `sources/templates/InHouse/InHouse.kicad_pcb`

It contains only a basic empty board shell, thickness, paper size, and setup values. It does not contain a populated rule table with editable board-specific rule entries, so KiCad has nothing to display in the rule editor UI.

## Correct way to edit the rules

After creating a real project from the InHouse template:

1. Open the PCB.
2. Go to `Board Setup`.
3. Open the relevant section (`Constraints`, `Design Rules`, `Net Classes`, `Track Widths`, `Via Dimensions`).
4. Adjust the values there and save.

This is the GUI workflow for a real project. The blank template itself is not a board with an editable rule table yet.

## Open question: why does the Design Rule Editor differ from the project defaults?

This is the key confusion: the `Design Rule Editor` and the project-level `design_settings` are not the same thing.

- `design_settings` in `InHouse.kicad_pro` are the default rule values KiCad applies to a new board created from the template.
- The `Design Rule Editor` shows the actual board-level rule table once the board is created and populated enough for KiCad to expose editable rules.

So the template can have valid DRC defaults without showing a populated editor in the blank board. The editor is a board-view UI, while the project JSON is a template-level configuration. This distinction is easy to miss, and it is the reason the same rules appear to "exist" in one place but not in the other.

## Practical consequence

The design rules are not "missing"; they are just stored as default project settings in the template rather than as visible editable board rules in the empty starter PCB. That is why the project behaves correctly in a real board while the empty template appears empty in the Design Rule Editor.
