# SVG Animation Reference

Non-obvious patterns and gotchas for animating SVG elements, especially in React/JSX projects with bundlers like Vite.

## Vite/Bundler Trap

**Never use `<style>` tags inside JSX-rendered SVGs.** Vite's `EnvironmentPluginContainer.transform` crashes with `Cannot read properties of undefined (reading 'call')`. SMIL elements (`<animate>`, `<animateTransform>`) work but run on CPU with no GPU acceleration.

**Solution:** Create a separate `.css` file, import it in the component, apply classes via `className`. This gets GPU compositing and avoids bundler issues.

## Splitting Opacity and Transform

When opacity and transform need different easing (common for assembly-line effects), split into separate `@keyframes` and stack them on one element:

```css
.element {
  animation:
    fade 14s linear infinite,        /* opacity: clean boundaries */
    move 14s cubic-bezier(0.12, 0, 1, 1) infinite;  /* transform: parabolic */
}
```

This is the key to making multi-phase animations feel right. Linear on opacity prevents weird fade artifacts, while the bezier on transform controls the motion feel.

## Hidden Phase Math

If a layer is active for 20% of its cycle, 80% is invisible. This ratio controls density:

- **Visible layers at any time** ≈ `(active% / 100) * (cycle_duration / stagger_interval)`
- To show 2-3 simultaneous layers: active phase ~20%, stagger ~2-3s on a 14-21s cycle
- If layers overlap at the same position, you have a z-ordering problem (see below)

## Negative Delays for Instant Start

Positive `animation-delay` creates a broken-looking ramp-up on page load where only some layers are active. Negate all delays so the animation appears already mid-cycle:

```
negative-delay = -(cycle-duration) + positive-delay
```

## Organic Stagger and Burst Pairs

Even spacing feels mechanical. Vary gaps for organic feel: `0, 1.6, 3.5, 5.0, 6.8` instead of `0, 2, 4, 6, 8`.

**Burst pairs:** Group elements in pairs (200-500ms apart) with larger gaps (~3-3.5s) between pairs. Creates an assembly-line pulse.

**Critical z-order rule for pairs:** The trailing element (later delay) must render *first* in DOM so the leading element (earlier delay, higher position) paints on top. SVG paints later DOM elements on top. This prevents the newer/lower layer from rendering over the older/higher one.

## Easing + Keyframe Interaction

**Use `linear` timing when you have many keyframe stops.** The motion curve is already baked into keyframe positions. A bezier on top causes double-easing/jitter.

**Use a bezier when you have few stops.** With 2-3 position keyframes, the bezier controls interpolation between them. `cubic-bezier(0.12, 0, 1, 1)` gives a parabolic O(n²) acceleration.

## SVG-Specific Techniques

### ViewBox clipping

The `viewBox` naturally clips content outside bounds. Calculate when an element's top point exits: if top is at `y=13` and viewBox starts at `y=0`, clip happens at `translateY = -13px`. Fade out *before* this point to avoid a hard cut.

### Box occlusion (emerge-from-container)

Make layers appear to rise from inside a container using SVG paint order instead of clipPath/mask:

1. Render container lid *before* animated layers (behind)
2. Render animated layers (start at positive Y, inside the box)
3. Render container walls *after* layers (on top, occluding layers while inside)

The walls naturally hide layers until they rise above the top edge. Add a subtle opacity fade-in (~200-400ms) during the emerge for a cleaner reveal. No mask/clip elements needed.

### Stagger spacing to prevent overlap

When layers share the same spawn position, ensure `stagger_interval > linger_duration` so a new layer never fades in while the previous one is still sitting at the same position. If they overlap spatially, no DOM order can fix the z-fighting since the correct stacking changes over time.

## SVG vs HTML Animation Differences

These are critical differences that will silently break your animations if you assume SVG elements behave like HTML elements.

### CSS `clip-path: inset()` does not work on SVG `<g>` elements

`clip-path: inset()` needs a reference bounding box. HTML elements have one. SVG `<g>` groups do NOT have an intrinsic box, so `inset()` percentages resolve to nothing. The clip has no visual effect.

**Apply clip-path to individual child elements** (rect, path, circle), not to groups. For grouped clipping, use SVG `<clipPath>` with a `<rect>` child, not CSS `clip-path`.

Exception: `<g>` elements containing `<path>` children sometimes work because the path establishes a bounding box. Test case by case, don't assume.

### CSS `transform` conflicts with SVG `transform` attribute

An SVG element with `<g transform="translate(100,50)">` cannot have an independent CSS `transform: scale(0.5)` animation. CSS `transform` and SVG `transform` occupy the same property slot. The CSS value replaces the SVG attribute, displacing the element entirely.

**Fix:** For elements positioned with SVG `transform` attributes (common with arc/pie slices, force-directed graph nodes), use `opacity`-only animations. Never add CSS transform animations to these elements.

### SVG elements have no consistent DOM wrapper structure

Different SVG mark types may render as:
- `<g class="mark-type"><rect>` (wrapped in a group)
- `<circle class="mark-type">` (bare element, no wrapper)
- `<text class="mark-type">` (bare element)

A CSS selector like `.mark-type circle` fails when the circle IS the `.mark-type` element. Always inspect the actual DOM structure before writing selectors. Use `circle.mark-type` (element selector) vs `.mark-type circle` (descendant selector) accordingly.

### Sequential chained animations need linear easing

When chaining animations so segment B starts when segment A ends, non-linear easing creates visible jitter at handoffs. Segment A decelerates, then segment B starts at full speed, producing a stutter.

**Fix:** Use `animation-timing-function: linear` for all segments in a chain. Reserve non-linear easing (spring, ease-out, bezier) for standalone animations that don't hand off to another element.
