# Chart Stories (Scrollytelling)

Scroll-driven narrative where one chart transforms through a sequence of states as the reader scrolls. Each step reframes the same base chart -- filter to a subset, highlight a series, swap the encoding, zoom the camera -- and the chart animates between states so nothing snaps.

**Use cases:** a "here's the whole picture, now let me walk you through it" explainer, revealing a trend step by step, comparing a protagonist against a field of grays one call-out at a time.

Prefer this over hand-rolled D3 scroll effects. The story API rides the same engine, transitions, theming, and a11y as a normal chart -- you write specs, not draw calls.

## Import

The story API is a vanilla subpath. It is not exported from the main barrel:

```js
import { createChartStory } from '@opendata-ai/openchart-vanilla/story';
```

There is no React/Vue/Svelte wrapper yet; drive it from vanilla inside your framework's mount hook.

## Model: base spec + cumulative patch steps

A story is a base `spec` plus an ordered list of `steps`. Each step is a **deep-partial patch** over the base, and patches are **cumulative**: step N is the base with patches `0..N` merged in order. So each step describes only what *changes* from the story's start, and everything accumulates as the reader scrolls forward.

```js
const story = createChartStory(container, {
  spec: {            // the base ChartSpec | LayerSpec | GraphSpec
    mark: 'line',
    data: [...],
    encoding: { x: {...}, y: {...}, color: {...} },
  },
  steps: [
    { spec: {} },                                   // step 0: the base, unchanged
    { highlight: ['Germany'] },                     // step 1: highlight one series
    { highlight: ['Germany', 'France'] },           // step 2: add another (cumulative)
    { spec: { transform: [{ filter: '...' }] } },   // step 3: filter the data
  ],
  triggerPosition: 0.4,   // viewport fraction where a step activates (default 0.4)
  cameraMode: 'step',     // 'step' (default) or 'scrub'
});
```

Step fields:

- `spec` -- a deep-partial spec patch (`StorySpecPatch`). Merged onto the accumulated spec.
- `highlight` -- `string[] | null`, sugar for `encoding.color.highlight` (the editorial "color these, gray the rest" move). Cleaner than writing the color patch by hand.
- `camera` -- optional per-step camera target (a data region or full view). Only applied when `cameraMode: 'step'`.

## Instance API

`createChartStory` returns a `ChartStoryInstance`:

- `registerStep(index, element)` -- associate a DOM element (a scroll section) with a step index. The built-in scroll driver activates the step when that element crosses `triggerPosition`.
- `setContainer(element)` -- register the scrolling container that wraps all the step elements.
- `goTo(index)` -- jump to a step programmatically (for buttons, tests, or a custom driver).
- `currentStep` (`-1` before any step activates), `totalSteps` -- read state.
- `destroy()` -- tear down the scroll driver, camera, and underlying chart instance (cancels any in-flight transition).

If you want your own scroll logic instead of the built-in driver, skip `registerStep` and call `goTo` yourself.

## Morph vs crossfade: the honesty rule

Between steps, the story picks the transition automatically, and **no step ever visibly snaps**:

- **Marks morph** (geometry interpolates -- bars grow, points slide, lines/areas path-morph) when the diff is data-only and passes the update-transition gate: same mark type, same encoding shape, just different data or highlight.
- **The whole chart crossfades** when the diff falls outside that gate: a mark-type change, a re-encode, an axis swap. Re-matching marks across those changes is intractable, so the story ghosts the current SVG, swaps underneath, and fades the ghost out.

Practical consequence: **design steps that stay within the morph gate when you want smooth mark motion.** If a step both filters the data *and* changes the mark type, you get a crossfade, not a morph. Split it into two steps (morph the filter, then crossfade the mark change) if the motion matters. Reduced-motion readers get instant swaps either way.

## Camera

`cameraMode: 'step'` (default) applies each step's `camera` target -- fit the viewBox to a data region to zoom into the part of the chart the step is about, or `FULL_VIEW` to pull back out. `cameraMode: 'scrub'` ties the camera to continuous scroll progress rather than discrete step boundaries. Data morphing itself is still discrete (it happens at step boundaries); scrub only affects the camera.

## Gotchas

- **Patches are cumulative, not independent.** To *undo* something a prior step set, a later step must patch it back explicitly -- there's no automatic reset between steps. Scrolling backward re-resolves from the base, so reverse scrolling is consistent.
- **`highlight: null` does not clear a prior highlight.** It maps to `undefined` in the patch, and a deep-merge skips `undefined`, so the accumulated highlight from an earlier step carries through. To actually de-highlight in a later step, pass an explicit `highlight: []`.
- **Vanilla only.** Inside React/Vue/Svelte, create the story in a mount effect and `destroy()` on cleanup; don't expect a `<ChartStory>` component.
