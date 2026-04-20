# SVG Asset Checklist

Tracking all SVG assets needed for the anatomical body map library.
Mark each as done when the SVG is created, reviewed, and placed in the correct directory.

## Naming Convention

```
svg/
├── full-body/
│   ├── male-front.svg
│   ├── male-back.svg
│   ├── female-front.svg
│   └── female-back.svg
└── parts/
    ├── head-neck/
    │   ├── head-neck-front.svg
    │   ├── head-neck-back.svg
    │   └── head-neck-lateral.svg
    ├── chest/
    │   └── chest-front.svg
    └── ... (etc.)
```

## SVG Requirements

- Format: SVG with individual `<path>` elements per clickable zone
- Each zone path must have a unique `id` attribute (e.g. `id="zone-upper-trapezius"`)
- Clean, no embedded raster images
- Musculoskeletal detail appropriate for clinical use
- Consistent art style across all assets
- Viewbox normalized for consistent scaling

---

## Layer 1: Full Body (4 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 1 | `full-body/male-front.svg` | Male full body, front view, ~10-12 major clickable regions | [ ] |
| 2 | `full-body/male-back.svg` | Male full body, back view, ~10-12 major clickable regions | [ ] |
| 3 | `full-body/female-front.svg` | Female full body, front view, ~10-12 major clickable regions | [ ] |
| 4 | `full-body/female-back.svg` | Female full body, back view, ~10-12 major clickable regions | [ ] |

## Layer 2: Zoomed Body Parts

### Head & Neck (3 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 5 | `parts/head-neck/head-neck-front.svg` | Head & neck front view with zones | [ ] |
| 6 | `parts/head-neck/head-neck-back.svg` | Head & neck back view with zones | [ ] |
| 7 | `parts/head-neck/head-neck-lateral.svg` | Head & neck side view with zones | [ ] |

### Chest (1 SVG)

| # | File | Description | Status |
|---|------|-------------|--------|
| 8 | `parts/chest/chest-front.svg` | Chest front view with muscle zones (pecs, serratus, etc.) | [ ] |

### Abdomen (1 SVG)

| # | File | Description | Status |
|---|------|-------------|--------|
| 9 | `parts/abdomen/abdomen-front.svg` | Abdomen front view with zones (upper/lower abs, obliques) | [ ] |

### Upper Back (1 SVG)

| # | File | Description | Status |
|---|------|-------------|--------|
| 10 | `parts/upper-back/upper-back-back.svg` | Upper back view with zones (traps, rhomboids, rear delts) | [ ] |

### Lower Back (2 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 11 | `parts/lower-back/lower-back-back.svg` | Lower back rear view with zones (erectors, QL) | [ ] |
| 12 | `parts/lower-back/lower-back-lateral.svg` | Lower back side view with zones | [ ] |

### Left Shoulder (3 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 13 | `parts/left-shoulder/left-shoulder-front.svg` | Left shoulder front view (anterior delt, clavicle area) | [ ] |
| 14 | `parts/left-shoulder/left-shoulder-back.svg` | Left shoulder back view (posterior delt, rotator cuff) | [ ] |
| 15 | `parts/left-shoulder/left-shoulder-lateral.svg` | Left shoulder side view (lateral delt, acromion) | [ ] |

### Right Shoulder (3 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 16 | `parts/right-shoulder/right-shoulder-front.svg` | Right shoulder front view | [ ] |
| 17 | `parts/right-shoulder/right-shoulder-back.svg` | Right shoulder back view | [ ] |
| 18 | `parts/right-shoulder/right-shoulder-lateral.svg` | Right shoulder side view | [ ] |

### Left Arm (4 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 19 | `parts/left-arm/left-arm-front.svg` | Left arm front view (biceps, forearm flexors) | [ ] |
| 20 | `parts/left-arm/left-arm-back.svg` | Left arm back view (triceps, forearm extensors) | [ ] |
| 21 | `parts/left-arm/left-arm-lateral.svg` | Left arm outer side view | [ ] |
| 22 | `parts/left-arm/left-arm-medial.svg` | Left arm inner side view | [ ] |

### Right Arm (4 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 23 | `parts/right-arm/right-arm-front.svg` | Right arm front view | [ ] |
| 24 | `parts/right-arm/right-arm-back.svg` | Right arm back view | [ ] |
| 25 | `parts/right-arm/right-arm-lateral.svg` | Right arm outer side view | [ ] |
| 26 | `parts/right-arm/right-arm-medial.svg` | Right arm inner side view | [ ] |

### Left Hand (2 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 27 | `parts/left-hand/left-hand-front.svg` | Left hand palm view with zones | [ ] |
| 28 | `parts/left-hand/left-hand-back.svg` | Left hand dorsal view with zones | [ ] |

### Right Hand (2 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 29 | `parts/right-hand/right-hand-front.svg` | Right hand palm view | [ ] |
| 30 | `parts/right-hand/right-hand-back.svg` | Right hand dorsal view | [ ] |

### Left Hip (3 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 31 | `parts/left-hip/left-hip-front.svg` | Left hip front view (hip flexors, TFL) | [ ] |
| 32 | `parts/left-hip/left-hip-back.svg` | Left hip back view (glutes, piriformis) | [ ] |
| 33 | `parts/left-hip/left-hip-lateral.svg` | Left hip side view (IT band, glute med) | [ ] |

### Right Hip (3 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 34 | `parts/right-hip/right-hip-front.svg` | Right hip front view | [ ] |
| 35 | `parts/right-hip/right-hip-back.svg` | Right hip back view | [ ] |
| 36 | `parts/right-hip/right-hip-lateral.svg` | Right hip side view | [ ] |

### Left Leg (4 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 37 | `parts/left-leg/left-leg-front.svg` | Left leg front view (quads, shin) | [ ] |
| 38 | `parts/left-leg/left-leg-back.svg` | Left leg back view (hamstrings, calves) | [ ] |
| 39 | `parts/left-leg/left-leg-lateral.svg` | Left leg outer side view | [ ] |
| 40 | `parts/left-leg/left-leg-medial.svg` | Left leg inner side view (adductors) | [ ] |

### Right Leg (4 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 41 | `parts/right-leg/right-leg-front.svg` | Right leg front view | [ ] |
| 42 | `parts/right-leg/right-leg-back.svg` | Right leg back view | [ ] |
| 43 | `parts/right-leg/right-leg-lateral.svg` | Right leg outer side view | [ ] |
| 44 | `parts/right-leg/right-leg-medial.svg` | Right leg inner side view | [ ] |

### Left Foot (4 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 45 | `parts/left-foot/left-foot-front.svg` | Left foot top (dorsal) view | [ ] |
| 46 | `parts/left-foot/left-foot-back.svg` | Left foot bottom (plantar) view | [ ] |
| 47 | `parts/left-foot/left-foot-lateral.svg` | Left foot outer side view | [ ] |
| 48 | `parts/left-foot/left-foot-medial.svg` | Left foot inner side view | [ ] |

### Right Foot (4 SVGs)

| # | File | Description | Status |
|---|------|-------------|--------|
| 49 | `parts/right-foot/right-foot-front.svg` | Right foot top (dorsal) view | [ ] |
| 50 | `parts/right-foot/right-foot-back.svg` | Right foot bottom (plantar) view | [ ] |
| 51 | `parts/right-foot/right-foot-lateral.svg` | Right foot outer side view | [ ] |
| 52 | `parts/right-foot/right-foot-medial.svg` | Right foot inner side view | [ ] |

---

## Summary

| Category | Count | Done |
|----------|-------|------|
| Full Body (Layer 1) | 4 | 0 |
| Head & Neck | 3 | 0 |
| Chest | 1 | 0 |
| Abdomen | 1 | 0 |
| Upper Back | 1 | 0 |
| Lower Back | 2 | 0 |
| Shoulders (L+R) | 6 | 0 |
| Arms (L+R) | 8 | 0 |
| Hands (L+R) | 4 | 0 |
| Hips (L+R) | 6 | 0 |
| Legs (L+R) | 8 | 0 |
| Feet (L+R) | 8 | 0 |
| **Total** | **52** | **0** |

## Tips for AI SVG Generation

- Generate each body part in isolation on a white/transparent background
- Ensure individual muscle/bone zones are separate closed paths, not merged
- Use consistent line weight and art style across all assets
- Include zone `id` attributes in each path for interactivity
- Keep viewBox consistent within the same body part (all views of "left arm" should use similar dimensions)
- Test that paths are clean and don't self-intersect
