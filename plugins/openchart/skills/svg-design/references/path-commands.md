# SVG Path Commands

The `d` attribute on `<path>` is the most powerful (and most complex) part of SVG. Every shape can be expressed as a path.

## Command Syntax Rules

- Uppercase letter = absolute coordinates (relative to viewBox origin 0,0)
- Lowercase letter = relative coordinates (offset from current cursor position)
- Commas between parameters are optional (whitespace works)
- Multiple coordinate pairs after a command repeat that command
- `Z`/`z` are identical (no parameters)

## Move To: M / m

Picks up the pen and moves without drawing. Every path starts with this.

```
M x y       -- move to absolute position
m dx dy     -- move relative to current position
```

```xml
<!-- Start at (10, 10) -->
<path d="M 10 10 ..." />
```

Multiple M commands in one path create **subpaths** (separate shapes within one `<path>` element). This is how you create compound shapes like a donut (outer circle + inner circle).

## Line To: L / l

Draw a straight line from current position to the target.

```
L x y       -- line to absolute position
l dx dy     -- line relative to current position
```

```xml
<!-- Diagonal line from (10,10) to (90,90) -->
<path d="M 10 10 L 90 90" stroke="black" fill="none" />
```

## Horizontal Line: H / h

Shorthand for horizontal lines. Only takes one parameter.

```
H x         -- horizontal line to absolute x
h dx        -- horizontal line relative
```

## Vertical Line: V / v

Shorthand for vertical lines. Only takes one parameter.

```
V y         -- vertical line to absolute y
v dy        -- vertical line relative
```

## Close Path: Z / z

Draws a straight line back to the most recent `M` point. No parameters.

```xml
<!-- Triangle -->
<path d="M 10 80 L 50 10 L 90 80 Z" />
```

The difference between `Z` and drawing a line back to the start: `Z` creates a proper **line join** at the closure point (respecting stroke-linejoin), while `L` back to start creates two overlapping line caps.

## Cubic Bezier: C / c

The workhorse curve. Two control points give full control over curve shape.

```
C x1 y1, x2 y2, x y      -- absolute
c dx1 dy1, dx2 dy2, dx dy -- relative
```

- `(x1, y1)` = control point for the start of the curve (determines departure angle)
- `(x2, y2)` = control point for the end of the curve (determines arrival angle)
- `(x, y)` = endpoint of the curve

```xml
<!-- S-curve -->
<path d="M 10 80 C 40 10, 65 10, 95 80" stroke="black" fill="none" />

<!-- Symmetric arch -->
<path d="M 10 80 C 10 10, 90 10, 90 80" stroke="black" fill="none" />
```

**Mental model:** The control points act like magnets pulling the curve toward them. The curve starts heading toward the first control point and arrives from the direction of the second.

## Smooth Cubic: S / s

Shorthand for chaining cubic beziers. The first control point is automatically reflected from the previous curve's second control point.

```
S x2 y2, x y       -- absolute
s dx2 dy2, dx dy    -- relative
```

Only makes sense after a `C` or `S` command. If used after anything else, the first control point equals the current position (degenerates to a quadratic-like curve).

```xml
<!-- Smooth wave using C then S -->
<path d="M 10 80 C 40 10, 65 10, 95 80 S 150 150, 180 80" stroke="black" fill="none" />
```

## Quadratic Bezier: Q / q

Simpler curve with one control point. The control point determines the slope at both endpoints.

```
Q x1 y1, x y       -- absolute
q dx1 dy1, dx dy    -- relative
```

```xml
<!-- Simple arch -->
<path d="M 10 80 Q 50 10, 90 80" stroke="black" fill="none" />
```

Quadratic curves are less flexible than cubic but easier to reason about. Good for simple arcs and rounded corners.

## Smooth Quadratic: T / t

Shorthand for chaining quadratics. Control point is auto-reflected.

```
T x y       -- absolute
t dx dy     -- relative
```

```xml
<!-- Sine-wave-like pattern -->
<path d="M 10 80 Q 52.5 10, 95 80 T 180 80" stroke="black" fill="none" />
```

## Arc: A / a

Draws an elliptical arc. The most parameter-heavy command.

```
A rx ry x-rotation large-arc-flag sweep-flag x y      -- absolute
a rx ry x-rotation large-arc-flag sweep-flag dx dy     -- relative
```

| Parameter | Meaning |
|-----------|---------|
| `rx` | Horizontal radius of the ellipse |
| `ry` | Vertical radius (same as rx for circles) |
| `x-rotation` | Rotation of the ellipse in degrees |
| `large-arc-flag` | `0` = smaller arc (<180 deg), `1` = larger arc (>180 deg) |
| `sweep-flag` | `0` = counter-clockwise, `1` = clockwise |
| `x y` | Endpoint of the arc |

Given two points and a radius, there are 4 possible arcs. The flags select which one:

```
large-arc=0, sweep=0  ->  small arc, counter-clockwise
large-arc=0, sweep=1  ->  small arc, clockwise
large-arc=1, sweep=0  ->  large arc, counter-clockwise
large-arc=1, sweep=1  ->  large arc, clockwise
```

```xml
<!-- Semicircle (half of a circle with radius 25) -->
<path d="M 10 50 A 25 25 0 0 1 60 50" stroke="black" fill="none" />

<!-- Full circle using two arcs -->
<path d="M 50 25 A 25 25 0 1 1 50 75 A 25 25 0 1 1 50 25 Z" />
```

## Hand-Writing Simple Paths

Common shapes expressed as paths:

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

## Path Optimization Tips

1. **Use relative commands** when drawing shapes. `l 0 10` is shorter than `L 45.5 67.3`
2. **Use H/V** instead of `L x y` when only one coordinate changes
3. **Use S** after C to chain smooth curves (saves 2 parameters per segment)
4. **Use T** after Q to chain smooth quadratics (saves 2 parameters per segment)
5. **Drop redundant whitespace**: `M10 10L20 20` is valid (letter acts as delimiter)
6. **Omit commas**: `C 10 20 30 40 50 60` works the same as `C 10,20 30,40 50,60`
7. **Chain implicit commands**: `M 0 0 10 10 20 0` is three move-tos. `L 10 10 20 20 30 30` is three line-tos
8. **Round coordinates** to 1-2 decimal places for icons
