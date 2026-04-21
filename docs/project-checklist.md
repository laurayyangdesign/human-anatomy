# Human Anatomy Body Map — Project Checklist

## Phase 1: Build the App (14 tasks with placeholder SVGs)

Follow the implementation plan at `docs/plans/2026-04-20-anatomical-body-map.md`.

- [ ] Task 1: Monorepo scaffolding (npm workspaces, tsconfig, .gitignore)
- [ ] Task 2: Core types (AnatomicalStructure, Zone, ZoneSelection, Severity, etc.)
- [ ] Task 3: Body part & region data (~13 body parts, views per part)
- [ ] Task 4: Muscle & bone data (55+ muscles, 30+ bones)
- [ ] Task 5: Search & taxonomy utilities (getById, getByRegion, search, getChildren)
- [ ] Task 6: React hooks (useSelection, useNavigation)
- [ ] Task 7: SeverityPopover component (Mild/Moderate/Severe card popover)
- [ ] Task 8: ViewTabs & ZoneOverlay components
- [ ] Task 9: Placeholder SVGs & SVG registry
- [ ] Task 10: FullBodyView component (Layer 1 — front+back side by side)
- [ ] Task 11: BodyPartView component (Layer 2 — zoomed view with tabs + Layer 3 — severity)
- [ ] Task 12: BodyMap main component (manages 3-layer navigation)
- [ ] Task 13: Demo app (playground to test everything)
- [ ] Task 14: Build & package verification (compile, test, push)

## Phase 2: Generate SVG Assets (50 files)

Use AI tools to generate all SVGs. Track progress in `docs/svg-asset-checklist.md`.

- [ ] Generate 2 full-body SVGs (front + back) for Layer 1
- [ ] Generate 48 zoomed body part SVGs for Layer 2
- [ ] Each SVG must have separate `<path>` elements per clickable zone with unique `id` attributes
- [ ] Consistent art style across all assets

## Phase 3: Integrate SVGs into the App

Send the generated SVGs to Claude. For each SVG:

- [ ] Extract zone `<path>` data from the SVG file
- [ ] Add zone definitions to `packages/core/src/data/zones.ts` (id, name, svgPathData, relatedStructures)
- [ ] Register the SVG in `packages/react/src/svg/index.ts`
- [ ] Replace placeholder components with real SVG renders
- [ ] Test in demo app — verify zones are clickable, severity popover works, colors apply

### Integration order (suggested — start with one body part to validate the flow):

- [ ] Upper back (1 SVG, simple test case)
- [ ] Full body front + back (Layer 1)
- [ ] Chest, abdomen
- [ ] Shoulders (L+R)
- [ ] Arms (L+R)
- [ ] Hips (L+R)
- [ ] Legs (L+R)
- [ ] Head & neck
- [ ] Lower back
- [ ] Hands (L+R)
- [ ] Feet (L+R)

## Phase 4: Final Verification

- [ ] All 50 SVGs integrated and zones mapped
- [ ] Layer 1 → Layer 2 → severity popover flow works end-to-end
- [ ] Multi-select across body parts works
- [ ] Selections persist when navigating back and forth
- [ ] Demo app fully functional
- [ ] npm packages build cleanly
- [ ] Push final version to GitHub
