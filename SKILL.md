---
name: beautify-simulink-diagram
description: Use when creating or improving the visual layout of a Simulink block diagram, including requests to beautify, tidy, align, space, group, or reroute a model. Apply readable layout conventions while preserving model behavior and connectivity.
---

# Simulink diagram visual conventions

Use these conventions as a readable default, not as a rigid template. Preserve a model's established layout when it is already clear, and follow explicit project or user conventions where they differ. Improve presentation without changing model behavior, connectivity, hierarchy, or execution order.

A diagram that is already clear is a good starting point. Make targeted refinements to it rather than redesigning it. A compact diagram is not automatically better: prefer mostly straight lines with few crossings, even if the layout is less compact.

**Never use automatic layout.** Do not call `Simulink.BlockDiagram.arrangeSystem` or any auto-arrange or autolayout feature, including the layout pass that runs after `model_edit`. Position every change explicitly, for example with `set_param(block, 'Position', ...)`, and keep the original layout as the baseline. Auto-arrange reshuffles every block and line route and is not an acceptable beautify step.

## Workflow

1. **Inspect the diagram.** Identify the model or subsystem in scope, its main signal paths, feedback paths, functional groups, and existing visual conventions. Preserve layout choices that already make the model clear.
2. **Choose the smallest useful layout change.** Decide the flow direction and the blocks or lines that need adjustment before editing. Prefer changes to block positions, line routing, and spacing over structural edits.
3. **Apply the layout.** Use supported Simulink editing tools or APIs. Keep the existing blocks, ports, connections, hierarchy, and execution order. Treat subsystem restructuring, changing block parameters, and changing signal connectivity as out of scope unless explicitly requested.
4. **Verify the result.** Check that the changed layout is clearer and that the model's blocks, port order, line endpoints and branches, hierarchy, and execution-relevant settings remain unchanged. Fix any regression before finishing.
5. **Use visual QA when available.** With the model loaded in a MATLAB session that supports display, export the diagram to a temporary PNG and inspect it. For example, `print(['-s' modelName], '-dpng', [tempname '.png'])` prints one model or subsystem diagram. MATLAB no-display mode disables Simulink printing; when it is unavailable, report that visual inspection was not performed and rely only on checks that were actually completed.

## Flow direction

- Prefer left-to-right signal flow: place sources and inputs toward the left, processing through the middle, and sinks or outputs toward the right.
- Keep related stages visually ordered and make the main path easy to follow at a glance.
- Use top-to-bottom flow when it better reflects the model's natural structure, such as a vertical sequence or a layout constrained by the available canvas. Do not force a direction that creates more crossings or obscures relationships.
- Route feedback paths around the main flow, commonly above or below it, and make the return path easy to trace.

## Placement and alignment

- Align blocks with related roles or stages, using consistent rows or columns and a shared visual grid where practical.
- Align blocks that sit near each other along a shared vertical grid, typically matching left edges (or centers) for a column of blocks. Most nearby blocks should align this way. Align a block only when it stays clear of other blocks and lines.
- Place related blocks near one another and keep the overall progression balanced; avoid arbitrary offsets and unnecessary zigzags.
- Preserve meaningful subsystem boundaries, port ordering, and existing spatial cues. Do not move blocks across groups or reorder them if that would make relationships less clear.

## Spacing and whitespace

- Leave enough room between blocks for readable signal lines, labels, and annotations. Keep spacing between comparable blocks reasonably consistent.
- Give functional groups and the overall diagram breathing room. Avoid both cramped layouts and large empty gaps that disconnect related elements.
- Use whitespace to separate stages and clarify hierarchy rather than filling every part of the canvas. Balance compactness against readability at the scale where the model will be viewed or printed.

## Signal routing and crossings

- Prefer direct, orthogonal routes with as few bends as practical. Keep signal lines mostly linear: a horizontal or vertical run should dominate each connection. Keep lines from running through blocks, labels, or unrelated groups.
- Resize blocks when that improves line formatting, for example making a block taller so its input and output ports line up with straight horizontal signals. Keep block size changes modest and consistent within a group.
- Minimize crossings, especially on the main signal path. Reroute a line when doing so improves clarity without making it excessively long or convoluted.
- When crossings cannot reasonably be avoided, keep them unambiguous and preserve Simulink's connection semantics. Do not alter connectivity merely to improve appearance.
- Keep feedback, control, and other secondary paths visually distinct from the main flow when useful, without relying on color alone.

## Grouping

- Keep functionally related blocks together and separate distinct stages or concerns with clear spacing.
- Use existing subsystems, areas, or other established grouping mechanisms when they clarify structure. Add grouping only when it helps navigation; avoid excessive nesting or decorative boundaries.
- Keep group labels concise and consistent. A group should describe its contents, not imply behavior that the model does not have.

### Subsystem hierarchy

- In a new model, keep short, straightforward logic visible at the parent level when seeing its steps and connections helps explain the model. Group a coherent functional stage when it has enough internal detail to distract from the parent-level flow, represents a meaningful stage or concern, or is reused.
- Make each subsystem boundary communicate a useful responsibility. Do not create a subsystem for every block, split a simple sequence into arbitrary pieces, or group unrelated logic just to fill out a hierarchy.
- Keep the hierarchy as shallow as practical: add a nested level only when it reveals a meaningful subfunction, and ensure each level helps readers understand either the overall flow or the subsystem's purpose.
- Name subsystems for their function or domain responsibility using concise, specific terms consistent with the model's terminology. Avoid generic names that do not distinguish the subsystem's role.
- Treat subsystem organization as a structural change, not automatically as a visual-only change. Preserve the intended behavior, signal connectivity, port order, execution order, and relevant subsystem semantics when grouping logic; do not introduce a boundary that changes behavior merely to improve readability.

## Labels and annotations

- Use concise, descriptive names for blocks, signals, groups, and model interfaces. Preserve domain terminology and existing naming conventions.
- Label signals when their meaning is not obvious from context, particularly at subsystem boundaries, branches, or important outputs. Avoid repeating information already clear from the block name or surrounding structure.
- Add annotations for useful context such as units, assumptions, or stage descriptions. Place them near the relevant content without obscuring blocks or lines, and distinguish explanatory text from executable model elements.
- Keep text readable at normal viewing and print sizes; avoid relying on color as the only way to convey meaning.

## Tradeoffs and exceptions

Readability and traceability take precedence over strict alignment, uniform spacing, or a single flow direction. Adapt the layout for feedback-heavy, multi-rate, stateful, physical, or otherwise non-linear models; a local exception is preferable to a misleading global arrangement. Work within canvas and print constraints, and favor the smallest layout change that makes the diagram clearer.
