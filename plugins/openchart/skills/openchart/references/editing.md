# Chart Editing

The editing system lets users drag, select, delete, and inline-edit chart elements. Pass an `onEdit` callback to activate edit mode. The library fires typed callbacks for all interactions but never mutates the spec. The consumer receives events and decides whether/how to apply changes.

## Activating Edit Mode

```tsx
// React -- edit mode is on when onEdit is defined
<Chart spec={spec} onEdit={handleEdit} />

// Toggle edit mode dynamically
<Chart spec={spec} onEdit={editing ? handleEdit : undefined} />
```

```typescript
// Vanilla JS
createChart(container, spec, { onEdit: handleEdit });
```

## Callback Architecture

There are two categories of callbacks: spec-modifying edits (flow through `onEdit`) and UI state callbacks (separate props).

### Spec-Modifying Edits (onEdit)

`onEdit` receives a discriminated union keyed by `type`:

```typescript
type ElementEdit =
  | { type: 'annotation';           annotation: TextAnnotation;    offset: AnnotationOffset }
  | { type: 'annotation-connector'; annotation: TextAnnotation;    endpoint: 'from' | 'to'; offset: AnnotationOffset }
  | { type: 'range-label';          annotation: RangeAnnotation;   labelOffset: AnnotationOffset }
  | { type: 'refline-label';        annotation: RefLineAnnotation; labelOffset: AnnotationOffset }
  | { type: 'chrome';               key: ChromeKey;                text: string; offset: AnnotationOffset }
  | { type: 'series-label';         series: string;                offset: AnnotationOffset }
  | { type: 'legend';               offset: AnnotationOffset }
  | { type: 'legend-toggle';        series: string;                hidden: boolean }
  | { type: 'delete';               element: ElementRef }
  | { type: 'text-edit';            element: ElementRef;           oldText: string; newText: string }

type AnnotationOffset = { dx?: number; dy?: number }
type ChromeKey = 'title' | 'subtitle' | 'source' | 'byline' | 'footer'
```

### UI State Callbacks (separate props)

These fire for selection and text editing interactions but don't imply spec changes:

```tsx
<Chart
  spec={spec}
  onEdit={handleEdit}
  onSelect={(element: ElementRef) => { /* element clicked */ }}
  onDeselect={(element: ElementRef) => { /* element deselected */ }}
  onTextEdit={(element: ElementRef, oldText: string, newText: string) => { /* text committed */ }}
/>
```

`onTextEdit` fires on both the UI state callback and through `onEdit` as `{ type: 'text-edit' }`. Handle it in whichever place makes sense for your architecture.

## ElementRef (Element Identity)

`ElementRef` is a discriminated union that identifies any selectable/editable element:

```typescript
type ElementRef =
  | { type: 'annotation';    index: number; id?: string }
  | { type: 'chrome';        key: ChromeKey }
  | { type: 'series-label';  series: string }
  | { type: 'legend' }
  | { type: 'legend-entry';  series: string; index: number }
```

Helper constructors avoid manually building these objects:

```typescript
import { elementRef } from '@opendata-ai/openchart-core';

elementRef.annotation(2)           // { type: 'annotation', index: 2 }
elementRef.annotation(2, 'my-id')  // { type: 'annotation', index: 2, id: 'my-id' }
elementRef.chrome('title')         // { type: 'chrome', key: 'title' }
elementRef.seriesLabel('Revenue')  // { type: 'series-label', series: 'Revenue' }
elementRef.legend()                // { type: 'legend' }
elementRef.legendEntry('Revenue', 0)  // { type: 'legend-entry', series: 'Revenue', index: 0 }
```

The optional `id` field on annotation refs provides stable identity across spec mutations (when annotations get reordered, added, or removed).

## Persisting Edits to Spec

Each edit gives you the new offset. Write it back into the corresponding spec field to persist the position.

| Edit type | Spec field to update |
| --- | --- |
| `annotation` | `annotation.offset` on the matching `TextAnnotation` |
| `annotation-connector` | `annotation.connectorOffset.from` or `.to` on the matching `TextAnnotation` |
| `range-label` | `annotation.labelOffset` on the matching `RangeAnnotation` |
| `refline-label` | `annotation.labelOffset` on the matching `RefLineAnnotation` |
| `chrome` | `chrome[key]` -- change from string to `{ text, offset }` |
| `series-label` | `labels.offsets[series]` |
| `legend` | `legend.offset` |

## Full React Example

```tsx
import { useState } from 'react';
import { Chart } from '@opendata-ai/openchart-react';
import type { ChartSpec, ElementEdit, TextAnnotation, RangeAnnotation, RefLineAnnotation } from '@opendata-ai/openchart-core';

function EditableChart() {
  const [spec, setSpec] = useState<ChartSpec>({
    mark: 'line',
    data: myData,
    encoding: { x: { field: 'date', type: 'temporal' }, y: { field: 'value', type: 'quantitative' } },
    chrome: {
      title: 'Revenue Growth Accelerates',
      subtitle: 'Quarterly revenue, 2022-2024 ($B)',
    },
    legend: { position: 'top' },
    labels: { density: 'auto' },
    annotations: [
      { type: 'text', x: '2024-Q4', y: 72, text: 'Record quarter', connector: true, offset: { dx: -80, dy: -20 } },
      { type: 'range', x1: '2024-Q1', x2: '2024-Q4', label: 'Growth phase', fill: '#6366f1', opacity: 0.08 },
      { type: 'refline', y: 50, label: 'Target: $50B', style: 'dashed' },
    ],
  });

  const handleEdit = (edit: ElementEdit) => {
    setSpec((prev) => {
      switch (edit.type) {
        case 'annotation':
          return {
            ...prev,
            annotations: prev.annotations!.map((a) =>
              a.type === 'text' && (a as TextAnnotation).text === edit.annotation.text
                ? { ...a, offset: edit.offset }
                : a
            ),
          };

        case 'annotation-connector':
          return {
            ...prev,
            annotations: prev.annotations!.map((a) => {
              if (a.type !== 'text' || (a as TextAnnotation).text !== edit.annotation.text) return a;
              const ta = a as TextAnnotation;
              return { ...ta, connectorOffset: { ...ta.connectorOffset, [edit.endpoint]: edit.offset } };
            }),
          };

        case 'range-label':
          return {
            ...prev,
            annotations: prev.annotations!.map((a) =>
              a.type === 'range' && (a as RangeAnnotation).label === edit.annotation.label
                ? { ...a, labelOffset: edit.labelOffset }
                : a
            ),
          };

        case 'refline-label':
          return {
            ...prev,
            annotations: prev.annotations!.map((a) =>
              a.type === 'refline' && (a as RefLineAnnotation).label === edit.annotation.label
                ? { ...a, labelOffset: edit.labelOffset }
                : a
            ),
          };

        case 'chrome':
          return {
            ...prev,
            chrome: { ...prev.chrome, [edit.key]: { text: edit.text, offset: edit.offset } },
          };

        case 'series-label':
          return {
            ...prev,
            labels: { ...prev.labels, offsets: { ...prev.labels?.offsets, [edit.series]: edit.offset } },
          };

        case 'legend':
          return { ...prev, legend: { ...prev.legend, offset: edit.offset } };
      }
    });
  };

  return <Chart spec={spec} onEdit={handleEdit} />;
}
```

This example covers drag repositioning. For deletion, text editing, and selection handling, see the full example below.

## Spec Fields for Persisted Offsets

After handling edits, the spec shape looks like this:

```typescript
{
  // Text annotation with persisted drag positions
  annotations: [
    {
      type: 'text',
      x: '2024-Q4',
      y: 72,
      text: 'Record quarter',
      connector: true,
      offset: { dx: -80, dy: -20 },           // annotation label position
      connectorOffset: {
        from: { dx: 5, dy: 0 },               // connector endpoint at the label
        to: { dx: 0, dy: -3 },                // connector endpoint at the data point
      },
    },
    {
      type: 'range',
      x1: '2024-Q1',
      x2: '2024-Q4',
      label: 'Growth phase',
      fill: '#6366f1',
      opacity: 0.08,
      labelOffset: { dx: 10, dy: 0 },         // range label position
    },
    {
      type: 'refline',
      y: 50,
      label: 'Target: $50B',
      style: 'dashed',
      labelOffset: { dx: -5, dy: 0 },         // refline label position
    },
  ],

  // Chrome elements with persisted offsets
  chrome: {
    title: { text: 'Revenue Growth Accelerates', offset: { dx: 0, dy: 0 } },
    subtitle: { text: 'Quarterly revenue, 2022-2024 ($B)', offset: { dx: 0, dy: 0 } },
  },

  // Series label offsets keyed by series name
  labels: {
    density: 'auto',
    offsets: {
      'Services': { dx: 12, dy: -4 },
      'Devices': { dx: 8, dy: 0 },
    },
  },

  // Legend position offset
  legend: {
    position: 'top',
    offset: { dx: 20, dy: 0 },
  },
}
```

## Selection API

Click an element to select it. The library renders selection overlays (handles, highlight ring) inside the SVG.

### Declarative (React)

```tsx
const [selected, setSelected] = useState<ElementRef | null>(null);

<Chart
  spec={spec}
  onEdit={handleEdit}
  selectedElement={selected ?? undefined}
  onSelect={setSelected}
  onDeselect={() => setSelected(null)}
/>
```

`selectedElement` is the controlled prop. Pass it to drive selection from external UI (property panels, annotation lists, etc.). Selection persists across spec updates.

### Imperative (Vanilla JS / React ref)

```typescript
// Vanilla JS
const chart = createChart(container, spec, { onEdit: handleEdit });
chart.select(elementRef.annotation(0));
chart.getSelectedElement();  // ElementRef | null
chart.deselect();

// React via ChartHandle ref
const chartRef = useRef<ChartHandle>(null);
<Chart ref={chartRef} spec={spec} onEdit={handleEdit} />

chartRef.current?.select(elementRef.chrome('title'));
chartRef.current?.getSelectedElement();
chartRef.current?.deselect();
```

## Text Editing

Double-click a text annotation or chrome element to edit it inline. The library renders a text input overlay inside the SVG.

- **Enter** commits the edit
- **Escape** cancels and restores original text

When committed, both `onTextEdit` (UI callback) and `onEdit({ type: 'text-edit' })` fire. Handle the spec update in your `onEdit` handler:

```typescript
case 'text-edit':
  if (edit.element.type === 'annotation') {
    return {
      ...prev,
      annotations: prev.annotations?.map((a, i) =>
        i === edit.element.index ? { ...a, text: edit.newText } : a
      ),
    };
  }
  if (edit.element.type === 'chrome') {
    const chrome = prev.chrome ?? {};
    const existing = chrome[edit.element.key];
    const updated = typeof existing === 'string'
      ? edit.newText
      : { ...existing, text: edit.newText };
    return { ...prev, chrome: { ...chrome, [edit.element.key]: updated } };
  }
  break;
```

## Deletion

Press Delete or Backspace with an element selected. The library fires `onEdit({ type: 'delete', element })` and the consumer removes it from the spec:

```typescript
case 'delete':
  if (edit.element.type === 'annotation') {
    return {
      ...prev,
      annotations: prev.annotations?.filter((_, i) => i !== edit.element.index),
    };
  }
  break;
```

## Legend Toggle

Click a legend entry to show/hide a series. Fires `onEdit({ type: 'legend-toggle', series, hidden })`. The consumer decides how to apply this (filter data, track hidden series in state, etc.).

## Keyboard Shortcuts

These work when the SVG has focus (click the chart first):

| Key | Action |
| --- | --- |
| Delete / Backspace | Delete selected element |
| Escape | Deselect current element, or cancel text edit |
| Tab | Cycle through editable elements |
| Enter | Enter text editing on selected text-editable element |

## Full React Example (with Selection, Deletion, Text Editing)

```tsx
import { useState, useRef } from 'react';
import { Chart } from '@opendata-ai/openchart-react';
import type { ChartSpec, ElementEdit, ElementRef, TextAnnotation, ChartHandle } from '@opendata-ai/openchart-core';

function EditableChart() {
  const [spec, setSpec] = useState<ChartSpec>(initialSpec);
  const [selected, setSelected] = useState<ElementRef | null>(null);
  const chartRef = useRef<ChartHandle>(null);

  const handleEdit = (edit: ElementEdit) => {
    setSpec((prev) => {
      switch (edit.type) {
        case 'delete':
          if (edit.element.type === 'annotation') {
            return {
              ...prev,
              annotations: prev.annotations?.filter((_, i) => i !== edit.element.index),
            };
          }
          return prev;

        case 'text-edit':
          if (edit.element.type === 'annotation') {
            return {
              ...prev,
              annotations: prev.annotations?.map((a, i) =>
                i === edit.element.index ? { ...a, text: edit.newText } : a
              ),
            };
          }
          if (edit.element.type === 'chrome') {
            const chrome = prev.chrome ?? {};
            const existing = chrome[edit.element.key];
            return {
              ...prev,
              chrome: {
                ...chrome,
                [edit.element.key]: typeof existing === 'string'
                  ? edit.newText
                  : { ...existing, text: edit.newText },
              },
            };
          }
          return prev;

        case 'annotation':
          return {
            ...prev,
            annotations: prev.annotations!.map((a) =>
              a.type === 'text' && (a as TextAnnotation).text === edit.annotation.text
                ? { ...a, offset: edit.offset }
                : a
            ),
          };

        case 'chrome':
          return {
            ...prev,
            chrome: { ...prev.chrome, [edit.key]: { text: edit.text, offset: edit.offset } },
          };

        case 'series-label':
          return {
            ...prev,
            labels: { ...prev.labels, offsets: { ...prev.labels?.offsets, [edit.series]: edit.offset } },
          };

        case 'legend':
          return { ...prev, legend: { ...prev.legend, offset: edit.offset } };

        // ... handle other edit types
        default:
          return prev;
      }
    });
  };

  return (
    <Chart
      ref={chartRef}
      spec={spec}
      onEdit={handleEdit}
      selectedElement={selected ?? undefined}
      onSelect={setSelected}
      onDeselect={() => setSelected(null)}
    />
  );
}
```

## Design Boundaries

The library and the consumer have clear ownership lines:

| Library owns (inside the SVG) | Consumer owns (outside the SVG) |
| --- | --- |
| Selection overlay, hover states | Toolbars, property panels, buttons |
| Text editing input overlay | Undo/redo (store spec snapshots) |
| Drag handles, cursor changes | Which edits to accept or reject |
| Keyboard focus management | External selection UI (lists, trees) |

All edits are callbacks, never mutations. The consumer is always in control.

## Notes

- **Edit mode activates automatically** when `onEdit` is defined -- no separate prop needed
- **Offsets are in pixels**, not data coordinates. They're relative to the element's natural position
- **`onAnnotationEdit` is the legacy API** for text annotations only. Prefer `onEdit` for all new integrations -- it covers all draggable elements
- **Selection persists across spec updates.** The library matches by ElementRef identity, so re-renders don't drop the selection
- **Saving to a backend**: serialize the updated spec as JSON after each edit, or debounce for performance
