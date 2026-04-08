# Evals — logo-icon-design

## Test 1 — Simple trigger
**Prompt**: "Create a logo for my SaaS B2B startup called Holosia"
**Expected**:
- [ ] Brief asked BEFORE generating SVG
- [ ] Creative direction proposed + validated
- [ ] Self-contained SVG (no external dependencies)
- [ ] Coherent B2B palette applied
- [ ] No anti-patterns (no generic purple gradient)

## Test 2 — UI Icon
**Prompt**: "Make me an icon to represent 'product scan' for a mobile app"
**Expected**:
- [ ] UI icon type detected (stroke-based, 24px grid)
- [ ] viewBox="0 0 24 24" used
- [ ] stroke-linecap: round applied
- [ ] Recognizable at real 16px
- [ ] Exportable functional SVG

## Test 3 — Chat environment
**Prompt**: "Simple logo for a personal project called Archilab"
**Expected**:
- [ ] SVG rendered inline (artifact/widget)
- [ ] Variants offered after delivery
- [ ] No reference to filesystem paths

## Test 4 — Lettermark + favicon
**Prompt**: "Lettermark BC + 32px favicon version"
**Expected**:
- [ ] Two SVGs produced: full lettermark + 32x32 simplified version
- [ ] Favicon mentally tested at small size
