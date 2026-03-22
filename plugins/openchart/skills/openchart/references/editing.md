# Chart Editing

The editing system lets users drag chart elements to reposition them. Pass an `onEdit` callback to activate edit mode. The callback fires a typed `ElementEdit` event for every drag.

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

## ElementEdit Type

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

type AnnotationOffset = { dx?: number; dy?: number }
type ChromeKey = 'title' | 'subtitle' | 'source' | 'byline' | 'footer'
```

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

## Notes

- **Edit mode activates automatically** when `onEdit` is defined -- no separate prop needed
- **Offsets are in pixels**, not data coordinates. They're relative to the element's natural position
- **`onAnnotationEdit` is the legacy API** for text annotations only. Prefer `onEdit` for all new integrations -- it covers all draggable elements
- **Saving to a backend**: serialize the updated spec as JSON after each edit, or debounce for performance
