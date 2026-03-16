# Path Patterns

## Arc Flag Combinations

Given two points and a radius, there are 4 possible arcs. The flags select which one:

```
A rx ry x-rotation large-arc-flag sweep-flag x y

large-arc=0, sweep=0  ->  small arc, counter-clockwise
large-arc=0, sweep=1  ->  small arc, clockwise
large-arc=1, sweep=0  ->  large arc, counter-clockwise
large-arc=1, sweep=1  ->  large arc, clockwise
```

## Hand-Written Path Templates (24x24 grid)

```xml
<!-- Square (10x10 at origin 2,2) -->
<path d="M 2 2 h 10 v 10 h -10 Z" />

<!-- Equilateral triangle centered roughly at 12,12 -->
<path d="M 12 4 L 20 20 L 4 20 Z" />

<!-- Plus/cross icon -->
<path d="M 12 5 v 14 M 5 12 h 14" />

<!-- Checkmark -->
<path d="M 5 12 l 4 4 l 8 -8" />

<!-- X mark -->
<path d="M 6 6 l 12 12 M 18 6 l -12 12" />

<!-- Circle (using arcs) -->
<path d="M 12 2 A 10 10 0 1 1 12 22 A 10 10 0 1 1 12 2 Z" />

<!-- Rounded rectangle (12x8 with 2px radius at 6,8) -->
<path d="M 8 8 h 8 a 2 2 0 0 1 2 2 v 4 a 2 2 0 0 1 -2 2 h -8 a 2 2 0 0 1 -2 -2 v -4 a 2 2 0 0 1 2 -2 Z" />

<!-- Heart -->
<path d="M 12 21 C 5 15 2 11 2 8 A 4 4 0 0 1 6 4 C 8 4 10 5.5 12 8 C 14 5.5 16 4 18 4 A 4 4 0 0 1 22 8 C 22 11 19 15 12 21 Z" />

<!-- Star (5-point) -->
<path d="M 12 2 l 3 7 h 7 l -5.5 4.5 l 2 7 L 12 16 l -6.5 4.5 l 2 -7 L 2 9 h 7 Z" />
```
