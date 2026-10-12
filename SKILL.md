# Simulink diagram visual conventions

Use these conventions as a readable default, not as a rigid template. Preserve a model's established layout when it is already clear, and follow explicit project or user conventions where they differ. Improve presentation without changing model behavior, connectivity, hierarchy, or execution order.

## Flow direction

- Prefer left-to-right signal flow: place sources and inputs toward the left, processing through the middle, and sinks or outputs toward the right.
- Keep related stages visually ordered and make the main path easy to follow at a glance.
- Use top-to-bottom flow when it better reflects the model's natural structure, such as a vertical sequence or a layout constrained by the available canvas. Do not force a direction that creates more crossings or obscures relationships.
- Route feedback paths around the main flow, commonly above or below it, and make the return path easy to trace.

## Placement and alignment

- Align blocks with related roles or stages, using consistent rows or columns and a shared visual grid where practical.
- Place related blocks near one another and keep the overall progression balanced; avoid arbitrary offsets and unnecessary zigzags.
- Preserve meaningful subsystem boundaries, port ordering, and existing spatial cues. Do not move blocks across groups or reorder them if that would make relationships less clear.

## Spacing and whitespace

- Leave enough room between blocks for readable signal lines, labels, and annotations. Keep spacing between comparable blocks reasonably consistent.
- Give functional groups and the overall diagram breathing room. Avoid both cramped layouts and large empty gaps that disconnect related elements.
- Use whitespace to separate stages and clarify hierarchy rather than filling every part of the canvas. Balance compactness against readability at the scale where the model will be viewed or printed.

## Signal routing and crossings

- Prefer direct, orthogonal routes with as few bends as practical. Keep lines from running through blocks, labels, or unrelated groups.
- Minimize crossings, especially on the main signal path. Reroute a line when doing so improves clarity without making it excessively long or convoluted.
- When crossings cannot reasonably be avoided, keep them unambiguous and preserve Simulink's connection semantics. Do not alter connectivity merely to improve appearance.
- Keep feedback, control, and other secondary paths visually distinct from the main flow when useful, without relying on color alone.

## Grouping

- Keep functionally related blocks together and separate distinct stages or concerns with clear spacing.
- Use existing subsystems, areas, or other established grouping mechanisms when they clarify structure. Add grouping only when it helps navigation; avoid excessive nesting or decorative boundaries.
- Keep group labels concise and consistent. A group should describe its contents, not imply behavior that the model does not have.

## Labels and annotations

- Use concise, descriptive names for blocks, signals, groups, and model interfaces. Preserve domain terminology and existing naming conventions.
- Label signals when their meaning is not obvious from context, particularly at subsystem boundaries, branches, or important outputs. Avoid repeating information already clear from the block name or surrounding structure.
- Add annotations for useful context such as units, assumptions, or stage descriptions. Place them near the relevant content without obscuring blocks or lines, and distinguish explanatory text from executable model elements.
- Keep text readable at normal viewing and print sizes; avoid relying on color as the only way to convey meaning.

## Tradeoffs and exceptions

Readability and traceability take precedence over strict alignment, uniform spacing, or a single flow direction. Adapt the layout for feedback-heavy, multi-rate, stateful, physical, or otherwise non-linear models; a local exception is preferable to a misleading global arrangement. Work within canvas and print constraints, and favor the smallest layout change that makes the diagram clearer.
