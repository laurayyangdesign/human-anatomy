# Anatomical Body Map Library — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a monorepo with two npm packages (`@anatomy/core` and `@anatomy/react`) that provide musculoskeletal anatomy data and an interactive SVG body map with 3-layer drill-down navigation and severity-based pain zone selection.

**Architecture:** Monorepo using npm workspaces. `@anatomy/core` is a pure TypeScript package with zero dependencies — it defines all anatomical structures (muscles, bones), body regions, taxonomy, and lookup utilities. `@anatomy/react` depends on `@anatomy/core` and provides React components: a `BodyMap` that handles Layer 1 (full body front+back), Layer 2 (zoomed body part with view tabs), and Layer 3 (severity popover on zone click). A demo app in `apps/demo` serves as a playground.

**Tech Stack:** TypeScript, React 18, Vite, Vitest, npm workspaces

**Spec:** `docs/specs/2026-04-19-anatomical-body-map-design.md`
**SVG Checklist:** `docs/svg-asset-checklist.md`

---

## File Map

### `@anatomy/core` (`packages/core/`)

| File | Responsibility |
|------|---------------|
| `src/types.ts` | All TypeScript interfaces and types (`AnatomicalStructure`, `BodySystem`, `BodyRegion`, `BodyPart`, `BodyPartView`, `Zone`, `ZoneSelection`, `Severity`, `Annotation`) |
| `src/data/regions.ts` | `BodyRegion` enum values and `BodyPart` definitions (~10-12 major parts) |
| `src/data/muscles.ts` | All muscle `AnatomicalStructure` entries with IDs, names, regions, hierarchy |
| `src/data/bones.ts` | All bone `AnatomicalStructure` entries with IDs, names, regions, hierarchy |
| `src/data/zones.ts` | Zone definitions per body part per view — the clickable zone map |
| `src/data/index.ts` | Re-exports all data |
| `src/taxonomy.ts` | Hierarchical tree utilities (`getChildren`, `getParent`, `getAncestors`) |
| `src/search.ts` | `search()`, `getById()`, `getByRegion()`, `getAllMuscles()`, `getAllBones()` |
| `src/index.ts` | Public API barrel export |
| `package.json` | Package config |
| `tsconfig.json` | TypeScript config |

### `@anatomy/react` (`packages/react/`)

| File | Responsibility |
|------|---------------|
| `src/components/BodyMap.tsx` | Main component — manages 3-layer navigation state, delegates to sub-components |
| `src/components/FullBodyView.tsx` | Layer 1 — renders front+back SVGs side by side with clickable body part regions |
| `src/components/BodyPartView.tsx` | Layer 2 — renders zoomed SVG of selected body part, view tabs, zone overlays |
| `src/components/SeverityPopover.tsx` | Layer 3 — card popover with Mild/Moderate/Severe options |
| `src/components/ZoneOverlay.tsx` | Renders clickable zone paths on top of a body part SVG |
| `src/components/ViewTabs.tsx` | Front/Back/Lateral/Medial tab switcher |
| `src/components/index.ts` | Re-exports all components |
| `src/hooks/useSelection.ts` | Selection state management (add/remove/update zone selections) |
| `src/hooks/useNavigation.ts` | Layer navigation state (current layer, body part, view) |
| `src/svg/index.ts` | SVG registry — maps body part + view to SVG imports |
| `src/svg/placeholders.tsx` | Placeholder SVGs for development (simple rectangles with labels) |
| `src/styles.css` | Component styles (popover, tabs, hover states, severity colors) |
| `src/index.ts` | Public API barrel export |
| `package.json` | Package config |
| `tsconfig.json` | TypeScript config |

### Demo App (`apps/demo/`)

| File | Responsibility |
|------|---------------|
| `index.html` | HTML entry point |
| `src/main.tsx` | React entry point |
| `src/App.tsx` | Demo app — renders BodyMap with selection display panel |
| `src/App.css` | Demo styling |
| `package.json` | Package config |
| `vite.config.ts` | Vite config |

### Root

| File | Responsibility |
|------|---------------|
| `package.json` | Workspace root with npm workspaces config |
| `tsconfig.json` | Base TypeScript config |

---

## Task 1: Monorepo Scaffolding

**Files:**
- Create: `package.json` (root)
- Create: `tsconfig.json` (root)
- Create: `packages/core/package.json`
- Create: `packages/core/tsconfig.json`
- Create: `packages/react/package.json`
- Create: `packages/react/tsconfig.json`
- Create: `apps/demo/package.json`
- Create: `apps/demo/vite.config.ts`
- Create: `apps/demo/index.html`
- Create: `.gitignore`

- [ ] **Step 1: Create root package.json with workspaces**

```json
{
  "name": "human-anatomy",
  "private": true,
  "workspaces": [
    "packages/*",
    "apps/*"
  ],
  "scripts": {
    "build": "npm run build --workspaces",
    "test": "npm run test --workspaces --if-present",
    "dev": "npm run dev --workspace=apps/demo"
  }
}
```

- [ ] **Step 2: Create root tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "jsx": "react-jsx"
  }
}
```

- [ ] **Step 3: Create packages/core/package.json**

```json
{
  "name": "@anatomy/core",
  "version": "0.1.0",
  "type": "module",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "vitest run"
  },
  "devDependencies": {
    "typescript": "^5.4.0",
    "vitest": "^2.0.0"
  }
}
```

- [ ] **Step 4: Create packages/core/tsconfig.json**

```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src"]
}
```

- [ ] **Step 5: Create packages/react/package.json**

```json
{
  "name": "@anatomy/react",
  "version": "0.1.0",
  "type": "module",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "vitest run"
  },
  "dependencies": {
    "@anatomy/core": "*"
  },
  "peerDependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "typescript": "^5.4.0",
    "vitest": "^2.0.0"
  }
}
```

- [ ] **Step 6: Create packages/react/tsconfig.json**

```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src"]
}
```

- [ ] **Step 7: Create apps/demo scaffolding**

`apps/demo/package.json`:
```json
{
  "name": "anatomy-demo",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "dependencies": {
    "@anatomy/core": "*",
    "@anatomy/react": "*",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "typescript": "^5.4.0",
    "vite": "^5.0.0"
  }
}
```

`apps/demo/vite.config.ts`:
```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
});
```

`apps/demo/index.html`:
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Anatomy Body Map Demo</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

- [ ] **Step 8: Create .gitignore**

```
node_modules/
dist/
.superpowers/
```

- [ ] **Step 9: Install dependencies**

Run: `npm install`
Expected: All workspaces linked, `node_modules/` created.

- [ ] **Step 10: Commit**

```bash
git add .
git commit -m "chore: scaffold monorepo with core, react, and demo packages"
```

---

## Task 2: Core Types

**Files:**
- Create: `packages/core/src/types.ts`
- Create: `packages/core/src/index.ts`
- Test: `packages/core/src/__tests__/types.test.ts`

- [ ] **Step 1: Write the test**

```ts
// packages/core/src/__tests__/types.test.ts
import { describe, it, expect } from "vitest";
import type {
  AnatomicalStructure,
  BodySystem,
  BodyRegion,
  BodyPart,
  BodyPartView,
  Zone,
  ZoneSelection,
  Severity,
  Annotation,
} from "../types";
import { SEVERITY_COLORS, BODY_SYSTEMS } from "../types";

describe("Core Types", () => {
  it("SEVERITY_COLORS has all three levels", () => {
    expect(SEVERITY_COLORS.mild).toBe("#FFD600");
    expect(SEVERITY_COLORS.moderate).toBe("#FF9800");
    expect(SEVERITY_COLORS.severe).toBe("#F44336");
  });

  it("BODY_SYSTEMS contains muscular and skeletal", () => {
    expect(BODY_SYSTEMS).toContain("muscular");
    expect(BODY_SYSTEMS).toContain("skeletal");
    expect(BODY_SYSTEMS).toHaveLength(2);
  });

  it("type-checks a valid AnatomicalStructure", () => {
    const structure: AnatomicalStructure = {
      id: "muscle.trapezius.upper",
      name: "Upper Trapezius",
      system: "muscular",
      region: "upper_back",
      parentId: "muscle.trapezius",
      aliases: ["upper traps"],
    };
    expect(structure.id).toBe("muscle.trapezius.upper");
    expect(structure.system).toBe("muscular");
  });

  it("type-checks a valid ZoneSelection", () => {
    const selection: ZoneSelection = {
      zoneId: "back.left_scapular",
      zoneName: "Left Scapular Zone",
      bodyPart: "upper_back",
      view: "back",
      severity: "moderate",
      relatedStructures: ["muscle.rhomboid_major"],
    };
    expect(selection.severity).toBe("moderate");
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd packages/core && npx vitest run src/__tests__/types.test.ts`
Expected: FAIL — module `../types` not found.

- [ ] **Step 3: Implement types.ts**

```ts
// packages/core/src/types.ts

export type BodySystem = "muscular" | "skeletal";

export const BODY_SYSTEMS: BodySystem[] = ["muscular", "skeletal"];

export type BodyRegion =
  | "head"
  | "neck"
  | "chest"
  | "abdomen"
  | "upper_back"
  | "lower_back"
  | "pelvis"
  | "left_shoulder"
  | "right_shoulder"
  | "left_upper_arm"
  | "right_upper_arm"
  | "left_forearm"
  | "right_forearm"
  | "left_hand"
  | "right_hand"
  | "left_hip"
  | "right_hip"
  | "left_thigh"
  | "right_thigh"
  | "left_shin"
  | "right_shin"
  | "left_foot"
  | "right_foot";

export interface AnatomicalStructure {
  id: string;
  name: string;
  system: BodySystem;
  region: BodyRegion;
  parentId?: string;
  aliases?: string[];
}

export interface BodyPart {
  id: string;
  name: string;
  side: "front" | "back" | "both";
}

export interface BodyPartView {
  bodyPartId: string;
  view: "front" | "back" | "lateral" | "medial";
  zones: Zone[];
}

export interface Zone {
  id: string;
  name: string;
  svgPathData: string;
  relatedStructures: string[];
}

export type Severity = "mild" | "moderate" | "severe";

export const SEVERITY_COLORS: Record<Severity, string> = {
  mild: "#FFD600",
  moderate: "#FF9800",
  severe: "#F44336",
};

export interface ZoneSelection {
  zoneId: string;
  zoneName: string;
  bodyPart: string;
  view: string;
  severity: Severity;
  relatedStructures: string[];
  annotation?: Annotation;
}

export interface Annotation {
  label?: string;
  notes?: string;
  color?: string;
  metadata?: Record<string, unknown>;
}
```

- [ ] **Step 4: Create index.ts barrel export**

```ts
// packages/core/src/index.ts
export * from "./types";
```

- [ ] **Step 5: Run test to verify it passes**

Run: `cd packages/core && npx vitest run src/__tests__/types.test.ts`
Expected: All tests PASS.

- [ ] **Step 6: Commit**

```bash
git add packages/core/src/
git commit -m "feat(core): add all TypeScript type definitions"
```

---

## Task 3: Body Part & Region Data

**Files:**
- Create: `packages/core/src/data/regions.ts`
- Create: `packages/core/src/data/index.ts`
- Test: `packages/core/src/__tests__/regions.test.ts`
- Modify: `packages/core/src/index.ts`

- [ ] **Step 1: Write the test**

```ts
// packages/core/src/__tests__/regions.test.ts
import { describe, it, expect } from "vitest";
import { BODY_PARTS, BODY_PART_VIEWS, getBodyPart, getViewsForBodyPart } from "../data/regions";

describe("Body Parts", () => {
  it("has 10-12 major body parts", () => {
    expect(BODY_PARTS.length).toBeGreaterThanOrEqual(10);
    expect(BODY_PARTS.length).toBeLessThanOrEqual(12);
  });

  it("each body part has id, name, and side", () => {
    for (const part of BODY_PARTS) {
      expect(part.id).toBeTruthy();
      expect(part.name).toBeTruthy();
      expect(["front", "back", "both"]).toContain(part.side);
    }
  });

  it("getBodyPart returns correct part by id", () => {
    const chest = getBodyPart("chest");
    expect(chest).toBeDefined();
    expect(chest!.name).toBe("Chest");
    expect(chest!.side).toBe("front");
  });

  it("getBodyPart returns undefined for unknown id", () => {
    expect(getBodyPart("nonexistent")).toBeUndefined();
  });

  it("getViewsForBodyPart returns views for a body part", () => {
    const backViews = getViewsForBodyPart("upper_back");
    expect(backViews.length).toBeGreaterThanOrEqual(1);
    expect(backViews.some((v) => v.view === "back")).toBe(true);
  });

  it("every body part has at least one view", () => {
    for (const part of BODY_PARTS) {
      const views = getViewsForBodyPart(part.id);
      expect(views.length).toBeGreaterThanOrEqual(1);
    }
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd packages/core && npx vitest run src/__tests__/regions.test.ts`
Expected: FAIL — module not found.

- [ ] **Step 3: Implement regions.ts**

```ts
// packages/core/src/data/regions.ts
import type { BodyPart, BodyPartView } from "../types";

export const BODY_PARTS: BodyPart[] = [
  { id: "head_neck", name: "Head & Neck", side: "both" },
  { id: "chest", name: "Chest", side: "front" },
  { id: "abdomen", name: "Abdomen", side: "front" },
  { id: "upper_back", name: "Upper Back", side: "back" },
  { id: "lower_back", name: "Lower Back", side: "back" },
  { id: "left_shoulder", name: "Left Shoulder", side: "both" },
  { id: "right_shoulder", name: "Right Shoulder", side: "both" },
  { id: "left_arm", name: "Left Arm", side: "both" },
  { id: "right_arm", name: "Right Arm", side: "both" },
  { id: "left_hip", name: "Left Hip", side: "both" },
  { id: "right_hip", name: "Right Hip", side: "both" },
  { id: "left_leg", name: "Left Leg", side: "both" },
  { id: "right_leg", name: "Right Leg", side: "both" },
];

// Views available per body part. Zones are empty initially —
// they get populated when real SVG assets are added.
export const BODY_PART_VIEWS: BodyPartView[] = [
  // Head & Neck
  { bodyPartId: "head_neck", view: "front", zones: [] },
  { bodyPartId: "head_neck", view: "back", zones: [] },
  { bodyPartId: "head_neck", view: "lateral", zones: [] },
  // Chest
  { bodyPartId: "chest", view: "front", zones: [] },
  // Abdomen
  { bodyPartId: "abdomen", view: "front", zones: [] },
  // Upper Back
  { bodyPartId: "upper_back", view: "back", zones: [] },
  // Lower Back
  { bodyPartId: "lower_back", view: "back", zones: [] },
  { bodyPartId: "lower_back", view: "lateral", zones: [] },
  // Left Shoulder
  { bodyPartId: "left_shoulder", view: "front", zones: [] },
  { bodyPartId: "left_shoulder", view: "back", zones: [] },
  { bodyPartId: "left_shoulder", view: "lateral", zones: [] },
  // Right Shoulder
  { bodyPartId: "right_shoulder", view: "front", zones: [] },
  { bodyPartId: "right_shoulder", view: "back", zones: [] },
  { bodyPartId: "right_shoulder", view: "lateral", zones: [] },
  // Left Arm
  { bodyPartId: "left_arm", view: "front", zones: [] },
  { bodyPartId: "left_arm", view: "back", zones: [] },
  { bodyPartId: "left_arm", view: "lateral", zones: [] },
  { bodyPartId: "left_arm", view: "medial", zones: [] },
  // Right Arm
  { bodyPartId: "right_arm", view: "front", zones: [] },
  { bodyPartId: "right_arm", view: "back", zones: [] },
  { bodyPartId: "right_arm", view: "lateral", zones: [] },
  { bodyPartId: "right_arm", view: "medial", zones: [] },
  // Left Hip
  { bodyPartId: "left_hip", view: "front", zones: [] },
  { bodyPartId: "left_hip", view: "back", zones: [] },
  { bodyPartId: "left_hip", view: "lateral", zones: [] },
  // Right Hip
  { bodyPartId: "right_hip", view: "front", zones: [] },
  { bodyPartId: "right_hip", view: "back", zones: [] },
  { bodyPartId: "right_hip", view: "lateral", zones: [] },
  // Left Leg
  { bodyPartId: "left_leg", view: "front", zones: [] },
  { bodyPartId: "left_leg", view: "back", zones: [] },
  { bodyPartId: "left_leg", view: "lateral", zones: [] },
  { bodyPartId: "left_leg", view: "medial", zones: [] },
  // Right Leg
  { bodyPartId: "right_leg", view: "front", zones: [] },
  { bodyPartId: "right_leg", view: "back", zones: [] },
  { bodyPartId: "right_leg", view: "lateral", zones: [] },
  { bodyPartId: "right_leg", view: "medial", zones: [] },
];

export function getBodyPart(id: string): BodyPart | undefined {
  return BODY_PARTS.find((p) => p.id === id);
}

export function getViewsForBodyPart(bodyPartId: string): BodyPartView[] {
  return BODY_PART_VIEWS.filter((v) => v.bodyPartId === bodyPartId);
}
```

Note: Hands and feet are omitted from the initial 13 body parts to stay within 10-12. They can be added later. The views have empty `zones` arrays — zones get populated when real SVG assets with mapped paths are added.

- [ ] **Step 4: Create data/index.ts and update core index.ts**

```ts
// packages/core/src/data/index.ts
export { BODY_PARTS, BODY_PART_VIEWS, getBodyPart, getViewsForBodyPart } from "./regions";
```

Update `packages/core/src/index.ts`:
```ts
export * from "./types";
export * from "./data";
```

- [ ] **Step 5: Run test to verify it passes**

Run: `cd packages/core && npx vitest run src/__tests__/regions.test.ts`
Expected: All tests PASS.

- [ ] **Step 6: Commit**

```bash
git add packages/core/
git commit -m "feat(core): add body part and region definitions"
```

---

## Task 4: Muscle & Bone Data

**Files:**
- Create: `packages/core/src/data/muscles.ts`
- Create: `packages/core/src/data/bones.ts`
- Modify: `packages/core/src/data/index.ts`
- Test: `packages/core/src/__tests__/anatomy-data.test.ts`

- [ ] **Step 1: Write the test**

```ts
// packages/core/src/__tests__/anatomy-data.test.ts
import { describe, it, expect } from "vitest";
import { MUSCLES } from "../data/muscles";
import { BONES } from "../data/bones";

describe("Muscles", () => {
  it("has at least 30 muscles defined", () => {
    expect(MUSCLES.length).toBeGreaterThanOrEqual(30);
  });

  it("every muscle has required fields", () => {
    for (const m of MUSCLES) {
      expect(m.id).toMatch(/^muscle\./);
      expect(m.name).toBeTruthy();
      expect(m.system).toBe("muscular");
      expect(m.region).toBeTruthy();
    }
  });

  it("muscle IDs are unique", () => {
    const ids = MUSCLES.map((m) => m.id);
    expect(new Set(ids).size).toBe(ids.length);
  });

  it("parent references point to existing muscles", () => {
    const ids = new Set(MUSCLES.map((m) => m.id));
    for (const m of MUSCLES) {
      if (m.parentId) {
        expect(ids.has(m.parentId)).toBe(true);
      }
    }
  });
});

describe("Bones", () => {
  it("has at least 20 bones defined", () => {
    expect(BONES.length).toBeGreaterThanOrEqual(20);
  });

  it("every bone has required fields", () => {
    for (const b of BONES) {
      expect(b.id).toMatch(/^bone\./);
      expect(b.name).toBeTruthy();
      expect(b.system).toBe("skeletal");
      expect(b.region).toBeTruthy();
    }
  });

  it("bone IDs are unique", () => {
    const ids = BONES.map((b) => b.id);
    expect(new Set(ids).size).toBe(ids.length);
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd packages/core && npx vitest run src/__tests__/anatomy-data.test.ts`
Expected: FAIL — modules not found.

- [ ] **Step 3: Implement muscles.ts**

```ts
// packages/core/src/data/muscles.ts
import type { AnatomicalStructure } from "../types";

export const MUSCLES: AnatomicalStructure[] = [
  // --- Head & Neck ---
  { id: "muscle.sternocleidomastoid", name: "Sternocleidomastoid", system: "muscular", region: "neck" },
  { id: "muscle.scalenes", name: "Scalenes", system: "muscular", region: "neck" },
  { id: "muscle.suboccipitals", name: "Suboccipitals", system: "muscular", region: "head" },

  // --- Chest ---
  { id: "muscle.pectoralis_major", name: "Pectoralis Major", system: "muscular", region: "chest" },
  { id: "muscle.pectoralis_major.upper", name: "Upper Pectoralis", system: "muscular", region: "chest", parentId: "muscle.pectoralis_major" },
  { id: "muscle.pectoralis_major.lower", name: "Lower Pectoralis", system: "muscular", region: "chest", parentId: "muscle.pectoralis_major" },
  { id: "muscle.pectoralis_minor", name: "Pectoralis Minor", system: "muscular", region: "chest" },
  { id: "muscle.serratus_anterior", name: "Serratus Anterior", system: "muscular", region: "chest" },

  // --- Abdomen ---
  { id: "muscle.rectus_abdominis", name: "Rectus Abdominis", system: "muscular", region: "abdomen" },
  { id: "muscle.external_oblique", name: "External Oblique", system: "muscular", region: "abdomen" },
  { id: "muscle.internal_oblique", name: "Internal Oblique", system: "muscular", region: "abdomen" },
  { id: "muscle.transverse_abdominis", name: "Transverse Abdominis", system: "muscular", region: "abdomen" },

  // --- Upper Back ---
  { id: "muscle.trapezius", name: "Trapezius", system: "muscular", region: "upper_back" },
  { id: "muscle.trapezius.upper", name: "Upper Trapezius", system: "muscular", region: "upper_back", parentId: "muscle.trapezius", aliases: ["upper traps"] },
  { id: "muscle.trapezius.middle", name: "Middle Trapezius", system: "muscular", region: "upper_back", parentId: "muscle.trapezius", aliases: ["mid traps"] },
  { id: "muscle.trapezius.lower", name: "Lower Trapezius", system: "muscular", region: "upper_back", parentId: "muscle.trapezius", aliases: ["lower traps"] },
  { id: "muscle.rhomboid_major", name: "Rhomboid Major", system: "muscular", region: "upper_back" },
  { id: "muscle.rhomboid_minor", name: "Rhomboid Minor", system: "muscular", region: "upper_back" },
  { id: "muscle.latissimus_dorsi", name: "Latissimus Dorsi", system: "muscular", region: "upper_back", aliases: ["lats"] },
  { id: "muscle.infraspinatus", name: "Infraspinatus", system: "muscular", region: "upper_back" },
  { id: "muscle.teres_major", name: "Teres Major", system: "muscular", region: "upper_back" },
  { id: "muscle.teres_minor", name: "Teres Minor", system: "muscular", region: "upper_back" },

  // --- Lower Back ---
  { id: "muscle.erector_spinae", name: "Erector Spinae", system: "muscular", region: "lower_back" },
  { id: "muscle.quadratus_lumborum", name: "Quadratus Lumborum", system: "muscular", region: "lower_back", aliases: ["QL"] },
  { id: "muscle.multifidus", name: "Multifidus", system: "muscular", region: "lower_back" },

  // --- Shoulder (left used as canonical, right mirrors) ---
  { id: "muscle.deltoid", name: "Deltoid", system: "muscular", region: "left_shoulder" },
  { id: "muscle.deltoid.anterior", name: "Anterior Deltoid", system: "muscular", region: "left_shoulder", parentId: "muscle.deltoid", aliases: ["front delt"] },
  { id: "muscle.deltoid.lateral", name: "Lateral Deltoid", system: "muscular", region: "left_shoulder", parentId: "muscle.deltoid", aliases: ["side delt"] },
  { id: "muscle.deltoid.posterior", name: "Posterior Deltoid", system: "muscular", region: "left_shoulder", parentId: "muscle.deltoid", aliases: ["rear delt"] },
  { id: "muscle.supraspinatus", name: "Supraspinatus", system: "muscular", region: "left_shoulder" },
  { id: "muscle.subscapularis", name: "Subscapularis", system: "muscular", region: "left_shoulder" },

  // --- Upper Arm ---
  { id: "muscle.biceps_brachii", name: "Biceps Brachii", system: "muscular", region: "left_upper_arm", aliases: ["biceps"] },
  { id: "muscle.triceps_brachii", name: "Triceps Brachii", system: "muscular", region: "left_upper_arm", aliases: ["triceps"] },
  { id: "muscle.brachialis", name: "Brachialis", system: "muscular", region: "left_upper_arm" },

  // --- Forearm ---
  { id: "muscle.brachioradialis", name: "Brachioradialis", system: "muscular", region: "left_forearm" },
  { id: "muscle.wrist_flexors", name: "Wrist Flexors", system: "muscular", region: "left_forearm" },
  { id: "muscle.wrist_extensors", name: "Wrist Extensors", system: "muscular", region: "left_forearm" },

  // --- Hip ---
  { id: "muscle.gluteus_maximus", name: "Gluteus Maximus", system: "muscular", region: "left_hip", aliases: ["glute max"] },
  { id: "muscle.gluteus_medius", name: "Gluteus Medius", system: "muscular", region: "left_hip", aliases: ["glute med"] },
  { id: "muscle.gluteus_minimus", name: "Gluteus Minimus", system: "muscular", region: "left_hip", aliases: ["glute min"] },
  { id: "muscle.piriformis", name: "Piriformis", system: "muscular", region: "left_hip" },
  { id: "muscle.iliopsoas", name: "Iliopsoas", system: "muscular", region: "left_hip", aliases: ["hip flexor"] },
  { id: "muscle.tensor_fasciae_latae", name: "Tensor Fasciae Latae", system: "muscular", region: "left_hip", aliases: ["TFL"] },

  // --- Thigh ---
  { id: "muscle.quadriceps", name: "Quadriceps", system: "muscular", region: "left_thigh", aliases: ["quads"] },
  { id: "muscle.quadriceps.rectus_femoris", name: "Rectus Femoris", system: "muscular", region: "left_thigh", parentId: "muscle.quadriceps" },
  { id: "muscle.quadriceps.vastus_lateralis", name: "Vastus Lateralis", system: "muscular", region: "left_thigh", parentId: "muscle.quadriceps" },
  { id: "muscle.quadriceps.vastus_medialis", name: "Vastus Medialis", system: "muscular", region: "left_thigh", parentId: "muscle.quadriceps" },
  { id: "muscle.hamstrings", name: "Hamstrings", system: "muscular", region: "left_thigh" },
  { id: "muscle.hamstrings.biceps_femoris", name: "Biceps Femoris", system: "muscular", region: "left_thigh", parentId: "muscle.hamstrings" },
  { id: "muscle.hamstrings.semimembranosus", name: "Semimembranosus", system: "muscular", region: "left_thigh", parentId: "muscle.hamstrings" },
  { id: "muscle.hamstrings.semitendinosus", name: "Semitendinosus", system: "muscular", region: "left_thigh", parentId: "muscle.hamstrings" },
  { id: "muscle.adductors", name: "Adductors", system: "muscular", region: "left_thigh" },
  { id: "muscle.it_band", name: "IT Band", system: "muscular", region: "left_thigh", aliases: ["iliotibial band"] },

  // --- Shin / Calf ---
  { id: "muscle.gastrocnemius", name: "Gastrocnemius", system: "muscular", region: "left_shin", aliases: ["calf"] },
  { id: "muscle.soleus", name: "Soleus", system: "muscular", region: "left_shin" },
  { id: "muscle.tibialis_anterior", name: "Tibialis Anterior", system: "muscular", region: "left_shin" },
  { id: "muscle.peroneals", name: "Peroneals", system: "muscular", region: "left_shin", aliases: ["fibularis"] },
];
```

- [ ] **Step 4: Implement bones.ts**

```ts
// packages/core/src/data/bones.ts
import type { AnatomicalStructure } from "../types";

export const BONES: AnatomicalStructure[] = [
  // --- Head & Neck ---
  { id: "bone.skull", name: "Skull", system: "skeletal", region: "head" },
  { id: "bone.cervical_spine", name: "Cervical Spine", system: "skeletal", region: "neck", aliases: ["C-spine"] },
  { id: "bone.mandible", name: "Mandible", system: "skeletal", region: "head", aliases: ["jaw"] },

  // --- Chest ---
  { id: "bone.sternum", name: "Sternum", system: "skeletal", region: "chest", aliases: ["breastbone"] },
  { id: "bone.ribs", name: "Ribs", system: "skeletal", region: "chest" },
  { id: "bone.clavicle", name: "Clavicle", system: "skeletal", region: "chest", aliases: ["collarbone"] },

  // --- Back ---
  { id: "bone.thoracic_spine", name: "Thoracic Spine", system: "skeletal", region: "upper_back", aliases: ["T-spine"] },
  { id: "bone.lumbar_spine", name: "Lumbar Spine", system: "skeletal", region: "lower_back", aliases: ["L-spine"] },
  { id: "bone.sacrum", name: "Sacrum", system: "skeletal", region: "lower_back" },
  { id: "bone.coccyx", name: "Coccyx", system: "skeletal", region: "lower_back", aliases: ["tailbone"] },

  // --- Shoulder ---
  { id: "bone.scapula", name: "Scapula", system: "skeletal", region: "left_shoulder", aliases: ["shoulder blade"] },
  { id: "bone.acromion", name: "Acromion", system: "skeletal", region: "left_shoulder", parentId: "bone.scapula" },

  // --- Arm ---
  { id: "bone.humerus", name: "Humerus", system: "skeletal", region: "left_upper_arm" },
  { id: "bone.radius", name: "Radius", system: "skeletal", region: "left_forearm" },
  { id: "bone.ulna", name: "Ulna", system: "skeletal", region: "left_forearm" },

  // --- Hand ---
  { id: "bone.carpals", name: "Carpals", system: "skeletal", region: "left_hand", aliases: ["wrist bones"] },
  { id: "bone.metacarpals", name: "Metacarpals", system: "skeletal", region: "left_hand" },
  { id: "bone.phalanges_hand", name: "Phalanges (Hand)", system: "skeletal", region: "left_hand", aliases: ["finger bones"] },

  // --- Hip / Pelvis ---
  { id: "bone.pelvis", name: "Pelvis", system: "skeletal", region: "pelvis" },
  { id: "bone.ilium", name: "Ilium", system: "skeletal", region: "left_hip", parentId: "bone.pelvis" },
  { id: "bone.ischium", name: "Ischium", system: "skeletal", region: "left_hip", parentId: "bone.pelvis", aliases: ["sit bone"] },

  // --- Leg ---
  { id: "bone.femur", name: "Femur", system: "skeletal", region: "left_thigh", aliases: ["thigh bone"] },
  { id: "bone.patella", name: "Patella", system: "skeletal", region: "left_thigh", aliases: ["kneecap"] },
  { id: "bone.tibia", name: "Tibia", system: "skeletal", region: "left_shin", aliases: ["shinbone"] },
  { id: "bone.fibula", name: "Fibula", system: "skeletal", region: "left_shin" },

  // --- Foot ---
  { id: "bone.tarsals", name: "Tarsals", system: "skeletal", region: "left_foot", aliases: ["ankle bones"] },
  { id: "bone.calcaneus", name: "Calcaneus", system: "skeletal", region: "left_foot", parentId: "bone.tarsals", aliases: ["heel bone"] },
  { id: "bone.metatarsals", name: "Metatarsals", system: "skeletal", region: "left_foot" },
  { id: "bone.phalanges_foot", name: "Phalanges (Foot)", system: "skeletal", region: "left_foot", aliases: ["toe bones"] },
];
```

- [ ] **Step 5: Update data/index.ts**

```ts
// packages/core/src/data/index.ts
export { BODY_PARTS, BODY_PART_VIEWS, getBodyPart, getViewsForBodyPart } from "./regions";
export { MUSCLES } from "./muscles";
export { BONES } from "./bones";
```

- [ ] **Step 6: Run test to verify it passes**

Run: `cd packages/core && npx vitest run src/__tests__/anatomy-data.test.ts`
Expected: All tests PASS.

- [ ] **Step 7: Commit**

```bash
git add packages/core/
git commit -m "feat(core): add muscle and bone anatomical data"
```

---

## Task 5: Search & Taxonomy Utilities

**Files:**
- Create: `packages/core/src/search.ts`
- Create: `packages/core/src/taxonomy.ts`
- Modify: `packages/core/src/index.ts`
- Test: `packages/core/src/__tests__/search.test.ts`
- Test: `packages/core/src/__tests__/taxonomy.test.ts`

- [ ] **Step 1: Write search test**

```ts
// packages/core/src/__tests__/search.test.ts
import { describe, it, expect } from "vitest";
import { getAllMuscles, getAllBones, getById, getByRegion, search } from "../search";

describe("search", () => {
  it("getAllMuscles returns only muscles", () => {
    const muscles = getAllMuscles();
    expect(muscles.length).toBeGreaterThan(0);
    expect(muscles.every((m) => m.system === "muscular")).toBe(true);
  });

  it("getAllBones returns only bones", () => {
    const bones = getAllBones();
    expect(bones.length).toBeGreaterThan(0);
    expect(bones.every((b) => b.system === "skeletal")).toBe(true);
  });

  it("getById finds a known structure", () => {
    const trap = getById("muscle.trapezius.upper");
    expect(trap).toBeDefined();
    expect(trap!.name).toBe("Upper Trapezius");
  });

  it("getById returns undefined for unknown id", () => {
    expect(getById("nonexistent")).toBeUndefined();
  });

  it("getByRegion returns structures in a region", () => {
    const chestStructures = getByRegion("chest");
    expect(chestStructures.length).toBeGreaterThan(0);
    expect(chestStructures.every((s) => s.region === "chest")).toBe(true);
  });

  it("search finds by name substring", () => {
    const results = search("trapezius");
    expect(results.length).toBeGreaterThan(0);
    expect(results.some((r) => r.id === "muscle.trapezius")).toBe(true);
  });

  it("search finds by alias", () => {
    const results = search("lats");
    expect(results.some((r) => r.id === "muscle.latissimus_dorsi")).toBe(true);
  });

  it("search is case-insensitive", () => {
    const results = search("BICEPS");
    expect(results.length).toBeGreaterThan(0);
  });
});
```

- [ ] **Step 2: Write taxonomy test**

```ts
// packages/core/src/__tests__/taxonomy.test.ts
import { describe, it, expect } from "vitest";
import { getChildren, getParent, getAncestors } from "../taxonomy";

describe("taxonomy", () => {
  it("getChildren returns child structures", () => {
    const trapChildren = getChildren("muscle.trapezius");
    expect(trapChildren.length).toBe(3);
    expect(trapChildren.map((c) => c.id)).toContain("muscle.trapezius.upper");
  });

  it("getChildren returns empty for leaf nodes", () => {
    expect(getChildren("muscle.trapezius.upper")).toHaveLength(0);
  });

  it("getParent returns parent structure", () => {
    const parent = getParent("muscle.trapezius.upper");
    expect(parent).toBeDefined();
    expect(parent!.id).toBe("muscle.trapezius");
  });

  it("getParent returns undefined for root structures", () => {
    expect(getParent("muscle.trapezius")).toBeUndefined();
  });

  it("getAncestors returns ancestor chain", () => {
    const ancestors = getAncestors("muscle.quadriceps.rectus_femoris");
    expect(ancestors.map((a) => a.id)).toContain("muscle.quadriceps");
  });
});
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `cd packages/core && npx vitest run src/__tests__/search.test.ts src/__tests__/taxonomy.test.ts`
Expected: FAIL — modules not found.

- [ ] **Step 4: Implement search.ts**

```ts
// packages/core/src/search.ts
import type { AnatomicalStructure, BodyRegion } from "./types";
import { MUSCLES } from "./data/muscles";
import { BONES } from "./data/bones";

const ALL_STRUCTURES: AnatomicalStructure[] = [...MUSCLES, ...BONES];

export function getAllMuscles(): AnatomicalStructure[] {
  return MUSCLES;
}

export function getAllBones(): AnatomicalStructure[] {
  return BONES;
}

export function getById(id: string): AnatomicalStructure | undefined {
  return ALL_STRUCTURES.find((s) => s.id === id);
}

export function getByRegion(region: BodyRegion): AnatomicalStructure[] {
  return ALL_STRUCTURES.filter((s) => s.region === region);
}

export function search(query: string): AnatomicalStructure[] {
  const q = query.toLowerCase();
  return ALL_STRUCTURES.filter(
    (s) =>
      s.name.toLowerCase().includes(q) ||
      s.id.toLowerCase().includes(q) ||
      (s.aliases ?? []).some((a) => a.toLowerCase().includes(q))
  );
}
```

- [ ] **Step 5: Implement taxonomy.ts**

```ts
// packages/core/src/taxonomy.ts
import type { AnatomicalStructure } from "./types";
import { MUSCLES } from "./data/muscles";
import { BONES } from "./data/bones";

const ALL_STRUCTURES: AnatomicalStructure[] = [...MUSCLES, ...BONES];

export function getChildren(parentId: string): AnatomicalStructure[] {
  return ALL_STRUCTURES.filter((s) => s.parentId === parentId);
}

export function getParent(id: string): AnatomicalStructure | undefined {
  const structure = ALL_STRUCTURES.find((s) => s.id === id);
  if (!structure?.parentId) return undefined;
  return ALL_STRUCTURES.find((s) => s.id === structure.parentId);
}

export function getAncestors(id: string): AnatomicalStructure[] {
  const ancestors: AnatomicalStructure[] = [];
  let current = ALL_STRUCTURES.find((s) => s.id === id);
  while (current?.parentId) {
    const parent = ALL_STRUCTURES.find((s) => s.id === current!.parentId);
    if (!parent) break;
    ancestors.push(parent);
    current = parent;
  }
  return ancestors;
}
```

- [ ] **Step 6: Update core index.ts**

```ts
// packages/core/src/index.ts
export * from "./types";
export * from "./data";
export { getAllMuscles, getAllBones, getById, getByRegion, search } from "./search";
export { getChildren, getParent, getAncestors } from "./taxonomy";
```

- [ ] **Step 7: Run tests to verify they pass**

Run: `cd packages/core && npx vitest run`
Expected: All tests PASS.

- [ ] **Step 8: Commit**

```bash
git add packages/core/
git commit -m "feat(core): add search and taxonomy utilities"
```

---

## Task 6: React Hooks — useSelection & useNavigation

**Files:**
- Create: `packages/react/src/hooks/useSelection.ts`
- Create: `packages/react/src/hooks/useNavigation.ts`
- Test: `packages/react/src/__tests__/useSelection.test.ts`
- Test: `packages/react/src/__tests__/useNavigation.test.ts`

- [ ] **Step 1: Write useSelection test**

```ts
// packages/react/src/__tests__/useSelection.test.ts
import { describe, it, expect } from "vitest";
import { renderHook, act } from "@testing-library/react";
import { useSelection } from "../hooks/useSelection";
import type { ZoneSelection } from "@anatomy/core";

describe("useSelection", () => {
  const mockSelection: ZoneSelection = {
    zoneId: "back.left_scapular",
    zoneName: "Left Scapular Zone",
    bodyPart: "upper_back",
    view: "back",
    severity: "moderate",
    relatedStructures: ["muscle.rhomboid_major"],
  };

  it("starts with empty selections", () => {
    const { result } = renderHook(() => useSelection());
    expect(result.current.selections).toEqual([]);
  });

  it("adds a selection", () => {
    const { result } = renderHook(() => useSelection());
    act(() => result.current.addSelection(mockSelection));
    expect(result.current.selections).toHaveLength(1);
    expect(result.current.selections[0].zoneId).toBe("back.left_scapular");
  });

  it("removes a selection by zoneId", () => {
    const { result } = renderHook(() => useSelection());
    act(() => result.current.addSelection(mockSelection));
    act(() => result.current.removeSelection("back.left_scapular"));
    expect(result.current.selections).toHaveLength(0);
  });

  it("updates severity of existing selection", () => {
    const { result } = renderHook(() => useSelection());
    act(() => result.current.addSelection(mockSelection));
    act(() => result.current.updateSeverity("back.left_scapular", "severe"));
    expect(result.current.selections[0].severity).toBe("severe");
  });

  it("replaces selection in single-select mode", () => {
    const { result } = renderHook(() => useSelection({ multiSelect: false }));
    act(() => result.current.addSelection(mockSelection));
    act(() =>
      result.current.addSelection({
        ...mockSelection,
        zoneId: "chest.left_pec",
        zoneName: "Left Pec",
      })
    );
    expect(result.current.selections).toHaveLength(1);
    expect(result.current.selections[0].zoneId).toBe("chest.left_pec");
  });

  it("clears all selections", () => {
    const { result } = renderHook(() => useSelection());
    act(() => result.current.addSelection(mockSelection));
    act(() => result.current.clearSelections());
    expect(result.current.selections).toHaveLength(0);
  });
});
```

- [ ] **Step 2: Write useNavigation test**

```ts
// packages/react/src/__tests__/useNavigation.test.ts
import { describe, it, expect } from "vitest";
import { renderHook, act } from "@testing-library/react";
import { useNavigation } from "../hooks/useNavigation";

describe("useNavigation", () => {
  it("starts at layer 1", () => {
    const { result } = renderHook(() => useNavigation());
    expect(result.current.layer).toBe(1);
    expect(result.current.selectedBodyPart).toBeNull();
    expect(result.current.selectedView).toBeNull();
  });

  it("navigates to layer 2 with a body part", () => {
    const { result } = renderHook(() => useNavigation());
    act(() => result.current.selectBodyPart("upper_back", "back"));
    expect(result.current.layer).toBe(2);
    expect(result.current.selectedBodyPart).toBe("upper_back");
    expect(result.current.selectedView).toBe("back");
  });

  it("switches view within layer 2", () => {
    const { result } = renderHook(() => useNavigation());
    act(() => result.current.selectBodyPart("left_arm", "front"));
    act(() => result.current.switchView("lateral"));
    expect(result.current.layer).toBe(2);
    expect(result.current.selectedView).toBe("lateral");
  });

  it("goes back to layer 1", () => {
    const { result } = renderHook(() => useNavigation());
    act(() => result.current.selectBodyPart("chest", "front"));
    act(() => result.current.goBack());
    expect(result.current.layer).toBe(1);
    expect(result.current.selectedBodyPart).toBeNull();
  });

  it("goBack from layer 1 does nothing", () => {
    const { result } = renderHook(() => useNavigation());
    act(() => result.current.goBack());
    expect(result.current.layer).toBe(1);
  });
});
```

- [ ] **Step 3: Run tests to verify they fail**

Run: `cd packages/react && npx vitest run`

Note: You will need to add `@testing-library/react` and `jsdom` to devDependencies first:
```bash
cd packages/react && npm install -D @testing-library/react jsdom
```
And add to `packages/react/vitest.config.ts`:
```ts
import { defineConfig } from "vitest/config";
export default defineConfig({ test: { environment: "jsdom" } });
```

Expected: FAIL — modules not found.

- [ ] **Step 4: Implement useSelection.ts**

```ts
// packages/react/src/hooks/useSelection.ts
import { useState, useCallback } from "react";
import type { ZoneSelection, Severity } from "@anatomy/core";

interface UseSelectionOptions {
  multiSelect?: boolean;
}

export function useSelection(options: UseSelectionOptions = {}) {
  const { multiSelect = true } = options;
  const [selections, setSelections] = useState<ZoneSelection[]>([]);

  const addSelection = useCallback(
    (selection: ZoneSelection) => {
      setSelections((prev) => {
        const without = prev.filter((s) => s.zoneId !== selection.zoneId);
        if (multiSelect) {
          return [...without, selection];
        }
        return [selection];
      });
    },
    [multiSelect]
  );

  const removeSelection = useCallback((zoneId: string) => {
    setSelections((prev) => prev.filter((s) => s.zoneId !== zoneId));
  }, []);

  const updateSeverity = useCallback((zoneId: string, severity: Severity) => {
    setSelections((prev) =>
      prev.map((s) => (s.zoneId === zoneId ? { ...s, severity } : s))
    );
  }, []);

  const clearSelections = useCallback(() => {
    setSelections([]);
  }, []);

  return { selections, addSelection, removeSelection, updateSeverity, clearSelections };
}
```

- [ ] **Step 5: Implement useNavigation.ts**

```ts
// packages/react/src/hooks/useNavigation.ts
import { useState, useCallback } from "react";

type ViewAngle = "front" | "back" | "lateral" | "medial";

interface NavigationState {
  layer: 1 | 2;
  selectedBodyPart: string | null;
  selectedView: ViewAngle | null;
}

export function useNavigation() {
  const [state, setState] = useState<NavigationState>({
    layer: 1,
    selectedBodyPart: null,
    selectedView: null,
  });

  const selectBodyPart = useCallback((bodyPartId: string, defaultView: ViewAngle) => {
    setState({ layer: 2, selectedBodyPart: bodyPartId, selectedView: defaultView });
  }, []);

  const switchView = useCallback((view: ViewAngle) => {
    setState((prev) => ({ ...prev, selectedView: view }));
  }, []);

  const goBack = useCallback(() => {
    setState((prev) => {
      if (prev.layer === 1) return prev;
      return { layer: 1, selectedBodyPart: null, selectedView: null };
    });
  }, []);

  return { ...state, selectBodyPart, switchView, goBack };
}
```

- [ ] **Step 6: Run tests to verify they pass**

Run: `cd packages/react && npx vitest run`
Expected: All tests PASS.

- [ ] **Step 7: Commit**

```bash
git add packages/react/
git commit -m "feat(react): add useSelection and useNavigation hooks"
```

---

## Task 7: SeverityPopover Component

**Files:**
- Create: `packages/react/src/components/SeverityPopover.tsx`
- Create: `packages/react/src/styles.css`
- Test: `packages/react/src/__tests__/SeverityPopover.test.tsx`

- [ ] **Step 1: Write the test**

```tsx
// packages/react/src/__tests__/SeverityPopover.test.tsx
import { describe, it, expect, vi } from "vitest";
import { render, screen, fireEvent } from "@testing-library/react";
import { SeverityPopover } from "../components/SeverityPopover";

describe("SeverityPopover", () => {
  it("renders zone name and three severity options", () => {
    render(
      <SeverityPopover
        zoneName="Left Scapular Zone"
        position={{ x: 100, y: 200 }}
        onSelect={vi.fn()}
        onDismiss={vi.fn()}
      />
    );
    expect(screen.getByText("Left Scapular Zone")).toBeTruthy();
    expect(screen.getByText("Mild")).toBeTruthy();
    expect(screen.getByText("Moderate")).toBeTruthy();
    expect(screen.getByText("Severe")).toBeTruthy();
  });

  it("calls onSelect with severity when option clicked", () => {
    const onSelect = vi.fn();
    render(
      <SeverityPopover
        zoneName="Test Zone"
        position={{ x: 0, y: 0 }}
        onSelect={onSelect}
        onDismiss={vi.fn()}
      />
    );
    fireEvent.click(screen.getByText("Severe"));
    expect(onSelect).toHaveBeenCalledWith("severe");
  });

  it("shows remove button when showRemove is true", () => {
    render(
      <SeverityPopover
        zoneName="Test Zone"
        position={{ x: 0, y: 0 }}
        onSelect={vi.fn()}
        onDismiss={vi.fn()}
        onRemove={vi.fn()}
        showRemove={true}
      />
    );
    expect(screen.getByText("Remove")).toBeTruthy();
  });

  it("calls onRemove when remove is clicked", () => {
    const onRemove = vi.fn();
    render(
      <SeverityPopover
        zoneName="Test Zone"
        position={{ x: 0, y: 0 }}
        onSelect={vi.fn()}
        onDismiss={vi.fn()}
        onRemove={onRemove}
        showRemove={true}
      />
    );
    fireEvent.click(screen.getByText("Remove"));
    expect(onRemove).toHaveBeenCalled();
  });
});
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd packages/react && npx vitest run src/__tests__/SeverityPopover.test.tsx`
Expected: FAIL — module not found.

- [ ] **Step 3: Implement SeverityPopover.tsx**

```tsx
// packages/react/src/components/SeverityPopover.tsx
import React from "react";
import { SEVERITY_COLORS, type Severity } from "@anatomy/core";

interface SeverityPopoverProps {
  zoneName: string;
  position: { x: number; y: number };
  onSelect: (severity: Severity) => void;
  onDismiss: () => void;
  onRemove?: () => void;
  showRemove?: boolean;
  severityColors?: Record<Severity, string>;
}

const SEVERITY_OPTIONS: { key: Severity; label: string }[] = [
  { key: "mild", label: "Mild" },
  { key: "moderate", label: "Moderate" },
  { key: "severe", label: "Severe" },
];

export function SeverityPopover({
  zoneName,
  position,
  onSelect,
  onDismiss,
  onRemove,
  showRemove = false,
  severityColors = SEVERITY_COLORS,
}: SeverityPopoverProps) {
  return (
    <>
      <div
        className="anatomy-popover-backdrop"
        onClick={onDismiss}
      />
      <div
        className="anatomy-severity-popover"
        style={{ left: position.x, top: position.y }}
      >
        <div className="anatomy-popover-header">
          <span className="anatomy-popover-label">Intensity Level</span>
          <span className="anatomy-popover-zone-name">{zoneName}</span>
        </div>
        <div className="anatomy-popover-options">
          {SEVERITY_OPTIONS.map(({ key, label }) => (
            <button
              key={key}
              className={`anatomy-severity-option anatomy-severity-${key}`}
              onClick={() => onSelect(key)}
            >
              <span
                className="anatomy-severity-dot"
                style={{ backgroundColor: severityColors[key] }}
              />
              <span>{label}</span>
            </button>
          ))}
        </div>
        {showRemove && onRemove && (
          <button className="anatomy-popover-remove" onClick={onRemove}>
            Remove
          </button>
        )}
      </div>
    </>
  );
}
```

- [ ] **Step 4: Create styles.css**

```css
/* packages/react/src/styles.css */

/* Popover backdrop */
.anatomy-popover-backdrop {
  position: fixed;
  inset: 0;
  z-index: 999;
}

/* Severity popover card */
.anatomy-severity-popover {
  position: absolute;
  z-index: 1000;
  background: #1e1e32;
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 12px;
  padding: 16px 20px;
  min-width: 200px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.4);
  color: #e0e0e0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.anatomy-popover-header {
  display: flex;
  flex-direction: column;
  gap: 2px;
  margin-bottom: 12px;
}

.anatomy-popover-label {
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: rgba(255, 255, 255, 0.5);
}

.anatomy-popover-zone-name {
  font-size: 15px;
  font-weight: 600;
}

.anatomy-popover-options {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.anatomy-severity-option {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 14px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  background: transparent;
  color: #e0e0e0;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s;
}

.anatomy-severity-option:hover {
  background: rgba(255, 255, 255, 0.08);
}

.anatomy-severity-mild { border-color: rgba(255, 214, 0, 0.3); }
.anatomy-severity-mild:hover { background: rgba(255, 214, 0, 0.1); }
.anatomy-severity-moderate { border-color: rgba(255, 152, 0, 0.3); }
.anatomy-severity-moderate:hover { background: rgba(255, 152, 0, 0.1); }
.anatomy-severity-severe { border-color: rgba(244, 67, 54, 0.3); }
.anatomy-severity-severe:hover { background: rgba(244, 67, 54, 0.1); }

.anatomy-severity-dot {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  flex-shrink: 0;
}

.anatomy-popover-remove {
  display: block;
  width: 100%;
  margin-top: 8px;
  padding: 8px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  background: transparent;
  color: rgba(255, 255, 255, 0.5);
  font-size: 13px;
  cursor: pointer;
}

.anatomy-popover-remove:hover {
  background: rgba(255, 60, 60, 0.1);
  color: #ff5370;
}

/* View Tabs */
.anatomy-view-tabs {
  display: flex;
  gap: 4px;
  margin-bottom: 12px;
}

.anatomy-view-tab {
  padding: 6px 14px;
  border-radius: 6px;
  border: none;
  background: rgba(255, 255, 255, 0.08);
  color: rgba(255, 255, 255, 0.6);
  font-size: 13px;
  cursor: pointer;
}

.anatomy-view-tab:hover {
  background: rgba(255, 255, 255, 0.12);
}

.anatomy-view-tab.active {
  background: #ffcb6b;
  color: #000;
  font-weight: 600;
}

/* Zone overlays */
.anatomy-zone {
  fill: transparent;
  stroke: transparent;
  cursor: pointer;
  transition: fill 0.15s, stroke 0.15s;
}

.anatomy-zone:hover {
  fill: rgba(255, 255, 255, 0.15);
  stroke: rgba(255, 255, 255, 0.4);
  stroke-width: 1.5;
}

/* Body map container */
.anatomy-body-map {
  position: relative;
  display: inline-block;
}

.anatomy-full-body {
  display: flex;
  gap: 24px;
  justify-content: center;
}

.anatomy-full-body svg {
  max-height: 500px;
  width: auto;
}

/* Back button */
.anatomy-back-button {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 6px;
  background: transparent;
  color: rgba(255, 255, 255, 0.7);
  font-size: 13px;
  cursor: pointer;
  margin-bottom: 12px;
}

.anatomy-back-button:hover {
  background: rgba(255, 255, 255, 0.08);
}
```

- [ ] **Step 5: Run test to verify it passes**

Run: `cd packages/react && npx vitest run src/__tests__/SeverityPopover.test.tsx`
Expected: All tests PASS.

- [ ] **Step 6: Commit**

```bash
git add packages/react/
git commit -m "feat(react): add SeverityPopover component and styles"
```

---

## Task 8: ViewTabs & ZoneOverlay Components

**Files:**
- Create: `packages/react/src/components/ViewTabs.tsx`
- Create: `packages/react/src/components/ZoneOverlay.tsx`
- Create: `packages/react/src/components/index.ts`

- [ ] **Step 1: Implement ViewTabs.tsx**

```tsx
// packages/react/src/components/ViewTabs.tsx
import React from "react";

type ViewAngle = "front" | "back" | "lateral" | "medial";

interface ViewTabsProps {
  views: ViewAngle[];
  activeView: ViewAngle;
  onViewChange: (view: ViewAngle) => void;
}

const VIEW_LABELS: Record<ViewAngle, string> = {
  front: "Front",
  back: "Back",
  lateral: "Lateral",
  medial: "Medial",
};

export function ViewTabs({ views, activeView, onViewChange }: ViewTabsProps) {
  return (
    <div className="anatomy-view-tabs">
      {views.map((view) => (
        <button
          key={view}
          className={`anatomy-view-tab ${view === activeView ? "active" : ""}`}
          onClick={() => onViewChange(view)}
        >
          {VIEW_LABELS[view]}
        </button>
      ))}
    </div>
  );
}
```

- [ ] **Step 2: Implement ZoneOverlay.tsx**

```tsx
// packages/react/src/components/ZoneOverlay.tsx
import React, { useState } from "react";
import type { Zone, Severity } from "@anatomy/core";
import { SEVERITY_COLORS } from "@anatomy/core";

interface ZoneOverlayProps {
  zones: Zone[];
  selectedZones: Map<string, Severity>;
  onZoneClick: (zone: Zone, event: React.MouseEvent) => void;
  severityColors?: Record<Severity, string>;
}

export function ZoneOverlay({
  zones,
  selectedZones,
  onZoneClick,
  severityColors = SEVERITY_COLORS,
}: ZoneOverlayProps) {
  const [hoveredZone, setHoveredZone] = useState<string | null>(null);

  return (
    <g className="anatomy-zone-overlay">
      {zones.map((zone) => {
        const severity = selectedZones.get(zone.id);
        const isSelected = severity !== undefined;
        const fillColor = isSelected
          ? severityColors[severity] + "66"
          : "transparent";
        const strokeColor = isSelected
          ? severityColors[severity]
          : "transparent";

        return (
          <path
            key={zone.id}
            d={zone.svgPathData}
            className="anatomy-zone"
            style={{
              fill: hoveredZone === zone.id && !isSelected
                ? "rgba(255,255,255,0.15)"
                : fillColor,
              stroke: strokeColor,
              strokeWidth: isSelected ? 2 : undefined,
            }}
            onClick={(e) => onZoneClick(zone, e)}
            onMouseEnter={() => setHoveredZone(zone.id)}
            onMouseLeave={() => setHoveredZone(null)}
          >
            <title>{zone.name}</title>
          </path>
        );
      })}
    </g>
  );
}
```

- [ ] **Step 3: Create components/index.ts**

```ts
// packages/react/src/components/index.ts
export { SeverityPopover } from "./SeverityPopover";
export { ViewTabs } from "./ViewTabs";
export { ZoneOverlay } from "./ZoneOverlay";
```

- [ ] **Step 4: Commit**

```bash
git add packages/react/src/components/
git commit -m "feat(react): add ViewTabs and ZoneOverlay components"
```

---

## Task 9: Placeholder SVGs & SVG Registry

**Files:**
- Create: `packages/react/src/svg/placeholders.tsx`
- Create: `packages/react/src/svg/index.ts`

- [ ] **Step 1: Create placeholder SVGs**

These are simple SVGs with labeled rectangles representing body parts. They allow full development and testing before real SVG assets are ready.

```tsx
// packages/react/src/svg/placeholders.tsx
import React from "react";
import { BODY_PARTS } from "@anatomy/core";
import type { BodyPart } from "@anatomy/core";

// Simple placeholder: body outline with labeled clickable regions
export function PlaceholderFullBody({
  side,
  onBodyPartClick,
}: {
  side: "front" | "back";
  onBodyPartClick: (bodyPartId: string) => void;
}) {
  const parts = BODY_PARTS.filter(
    (p) => p.side === side || p.side === "both"
  );

  // Layout body parts vertically in a simple diagram
  const LAYOUT: Record<string, { x: number; y: number; w: number; h: number }> = {
    head_neck:       { x: 130, y: 10,  w: 60, h: 50 },
    chest:           { x: 110, y: 70,  w: 100, h: 70 },
    abdomen:         { x: 120, y: 145, w: 80, h: 60 },
    upper_back:      { x: 110, y: 70,  w: 100, h: 70 },
    lower_back:      { x: 120, y: 145, w: 80, h: 60 },
    left_shoulder:   { x: 60,  y: 70,  w: 45, h: 40 },
    right_shoulder:  { x: 215, y: 70,  w: 45, h: 40 },
    left_arm:        { x: 40,  y: 115, w: 40, h: 100 },
    right_arm:       { x: 240, y: 115, w: 40, h: 100 },
    left_hip:        { x: 110, y: 210, w: 50, h: 50 },
    right_hip:       { x: 160, y: 210, w: 50, h: 50 },
    left_leg:        { x: 110, y: 265, w: 45, h: 130 },
    right_leg:       { x: 165, y: 265, w: 45, h: 130 },
  };

  return (
    <svg viewBox="0 0 320 420" width="280" xmlns="http://www.w3.org/2000/svg">
      <text x="160" y="415" textAnchor="middle" fill="#888" fontSize="12">
        {side === "front" ? "Front" : "Back"}
      </text>
      {parts.map((part) => {
        const pos = LAYOUT[part.id];
        if (!pos) return null;
        return (
          <g
            key={part.id}
            onClick={() => onBodyPartClick(part.id)}
            style={{ cursor: "pointer" }}
          >
            <rect
              x={pos.x} y={pos.y} width={pos.w} height={pos.h}
              rx={6}
              fill="rgba(130,170,255,0.1)"
              stroke="rgba(130,170,255,0.4)"
              strokeWidth={1.5}
            />
            <text
              x={pos.x + pos.w / 2}
              y={pos.y + pos.h / 2 + 4}
              textAnchor="middle"
              fill="#82aaff"
              fontSize="10"
            >
              {part.name}
            </text>
          </g>
        );
      })}
    </svg>
  );
}

export function PlaceholderBodyPart({
  bodyPartId,
  view,
}: {
  bodyPartId: string;
  view: string;
}) {
  return (
    <svg viewBox="0 0 300 400" width="300" xmlns="http://www.w3.org/2000/svg">
      <rect
        x={20} y={20} width={260} height={360}
        rx={12}
        fill="rgba(130,170,255,0.05)"
        stroke="rgba(130,170,255,0.3)"
        strokeWidth={1}
        strokeDasharray="6 3"
      />
      <text x={150} y={190} textAnchor="middle" fill="#82aaff" fontSize="16">
        {bodyPartId}
      </text>
      <text x={150} y={215} textAnchor="middle" fill="#666" fontSize="12">
        {view} view
      </text>
      <text x={150} y={245} textAnchor="middle" fill="#555" fontSize="11">
        (placeholder — add real SVG)
      </text>
    </svg>
  );
}
```

- [ ] **Step 2: Create SVG registry**

```ts
// packages/react/src/svg/index.ts

// Registry maps bodyPartId + view to SVG component or path.
// When real SVGs are added, import and register them here.
// For now, all lookups fall back to placeholders.

type ViewAngle = "front" | "back" | "lateral" | "medial";

interface SvgEntry {
  component?: React.ComponentType;
  path?: string;
}

const registry = new Map<string, SvgEntry>();

function registryKey(bodyPartId: string, view: ViewAngle): string {
  return `${bodyPartId}:${view}`;
}

export function registerSvg(
  bodyPartId: string,
  view: ViewAngle,
  entry: SvgEntry
): void {
  registry.set(registryKey(bodyPartId, view), entry);
}

export function getSvgEntry(
  bodyPartId: string,
  view: ViewAngle
): SvgEntry | undefined {
  return registry.get(registryKey(bodyPartId, view));
}

export function hasSvg(bodyPartId: string, view: ViewAngle): boolean {
  return registry.has(registryKey(bodyPartId, view));
}
```

- [ ] **Step 3: Commit**

```bash
git add packages/react/src/svg/
git commit -m "feat(react): add placeholder SVGs and SVG registry"
```

---

## Task 10: FullBodyView Component (Layer 1)

**Files:**
- Create: `packages/react/src/components/FullBodyView.tsx`
- Modify: `packages/react/src/components/index.ts`

- [ ] **Step 1: Implement FullBodyView.tsx**

```tsx
// packages/react/src/components/FullBodyView.tsx
import React from "react";
import { PlaceholderFullBody } from "../svg/placeholders";
import type { ZoneSelection } from "@anatomy/core";

type ViewAngle = "front" | "back" | "lateral" | "medial";

interface FullBodyViewProps {
  onBodyPartClick: (bodyPartId: string, defaultView: ViewAngle) => void;
  selections: ZoneSelection[];
}

export function FullBodyView({ onBodyPartClick, selections }: FullBodyViewProps) {
  // Count selections per body part to show indicators
  const selectionCounts = new Map<string, number>();
  for (const s of selections) {
    selectionCounts.set(s.bodyPart, (selectionCounts.get(s.bodyPart) ?? 0) + 1);
  }

  const handleClick = (side: "front" | "back") => (bodyPartId: string) => {
    onBodyPartClick(bodyPartId, side);
  };

  return (
    <div className="anatomy-full-body">
      <PlaceholderFullBody side="front" onBodyPartClick={handleClick("front")} />
      <PlaceholderFullBody side="back" onBodyPartClick={handleClick("back")} />
    </div>
  );
}
```

- [ ] **Step 2: Update components/index.ts**

Add to `packages/react/src/components/index.ts`:
```ts
export { FullBodyView } from "./FullBodyView";
```

- [ ] **Step 3: Commit**

```bash
git add packages/react/src/components/
git commit -m "feat(react): add FullBodyView component (Layer 1)"
```

---

## Task 11: BodyPartView Component (Layer 2)

**Files:**
- Create: `packages/react/src/components/BodyPartView.tsx`
- Modify: `packages/react/src/components/index.ts`

- [ ] **Step 1: Implement BodyPartView.tsx**

```tsx
// packages/react/src/components/BodyPartView.tsx
import React, { useState } from "react";
import { getViewsForBodyPart, getBodyPart } from "@anatomy/core";
import type { Zone, Severity, ZoneSelection } from "@anatomy/core";
import { ViewTabs } from "./ViewTabs";
import { ZoneOverlay } from "./ZoneOverlay";
import { SeverityPopover } from "./SeverityPopover";
import { PlaceholderBodyPart } from "../svg/placeholders";

type ViewAngle = "front" | "back" | "lateral" | "medial";

interface BodyPartViewProps {
  bodyPartId: string;
  initialView: ViewAngle;
  selections: ZoneSelection[];
  onZoneSelect: (selection: ZoneSelection) => void;
  onZoneDeselect: (zoneId: string) => void;
  onBack: () => void;
  severityColors?: Record<Severity, string>;
}

export function BodyPartView({
  bodyPartId,
  initialView,
  selections,
  onZoneSelect,
  onZoneDeselect,
  onBack,
  severityColors,
}: BodyPartViewProps) {
  const [currentView, setCurrentView] = useState<ViewAngle>(initialView);
  const [popover, setPopover] = useState<{
    zone: Zone;
    position: { x: number; y: number };
  } | null>(null);

  const bodyPart = getBodyPart(bodyPartId);
  const views = getViewsForBodyPart(bodyPartId);
  const availableViews = views.map((v) => v.view) as ViewAngle[];
  const currentViewData = views.find((v) => v.view === currentView);

  // Build a map of selected zones for this body part
  const selectedZoneMap = new Map<string, Severity>();
  for (const s of selections) {
    if (s.bodyPart === bodyPartId) {
      selectedZoneMap.set(s.zoneId, s.severity);
    }
  }

  const handleZoneClick = (zone: Zone, event: React.MouseEvent) => {
    const rect = (event.target as Element).getBoundingClientRect();
    const container = (event.target as Element).closest(".anatomy-body-map");
    const containerRect = container?.getBoundingClientRect() ?? { left: 0, top: 0 };

    setPopover({
      zone,
      position: {
        x: rect.left - containerRect.left + rect.width / 2,
        y: rect.top - containerRect.top + rect.height,
      },
    });
  };

  const handleSeveritySelect = (severity: Severity) => {
    if (!popover) return;
    onZoneSelect({
      zoneId: popover.zone.id,
      zoneName: popover.zone.name,
      bodyPart: bodyPartId,
      view: currentView,
      severity,
      relatedStructures: popover.zone.relatedStructures,
    });
    setPopover(null);
  };

  const handleRemove = () => {
    if (!popover) return;
    onZoneDeselect(popover.zone.id);
    setPopover(null);
  };

  const isReselect = popover ? selectedZoneMap.has(popover.zone.id) : false;

  return (
    <div>
      <button className="anatomy-back-button" onClick={onBack}>
        ← {bodyPart?.name ?? bodyPartId}
      </button>

      {availableViews.length > 1 && (
        <ViewTabs
          views={availableViews}
          activeView={currentView}
          onViewChange={setCurrentView}
        />
      )}

      <div style={{ position: "relative" }}>
        <PlaceholderBodyPart bodyPartId={bodyPartId} view={currentView} />

        {currentViewData && currentViewData.zones.length > 0 && (
          <svg
            style={{ position: "absolute", inset: 0, width: "100%", height: "100%" }}
            viewBox="0 0 300 400"
          >
            <ZoneOverlay
              zones={currentViewData.zones}
              selectedZones={selectedZoneMap}
              onZoneClick={handleZoneClick}
              severityColors={severityColors}
            />
          </svg>
        )}

        {popover && (
          <SeverityPopover
            zoneName={popover.zone.name}
            position={popover.position}
            onSelect={handleSeveritySelect}
            onDismiss={() => setPopover(null)}
            onRemove={handleRemove}
            showRemove={isReselect}
            severityColors={severityColors}
          />
        )}
      </div>
    </div>
  );
}
```

- [ ] **Step 2: Update components/index.ts**

Add to `packages/react/src/components/index.ts`:
```ts
export { BodyPartView } from "./BodyPartView";
```

- [ ] **Step 3: Commit**

```bash
git add packages/react/src/components/
git commit -m "feat(react): add BodyPartView component (Layer 2 + Layer 3)"
```

---

## Task 12: BodyMap Main Component

**Files:**
- Create: `packages/react/src/components/BodyMap.tsx`
- Create: `packages/react/src/index.ts`
- Modify: `packages/react/src/components/index.ts`

- [ ] **Step 1: Implement BodyMap.tsx**

```tsx
// packages/react/src/components/BodyMap.tsx
import React from "react";
import type { ZoneSelection, Severity } from "@anatomy/core";
import { useNavigation } from "../hooks/useNavigation";
import { FullBodyView } from "./FullBodyView";
import { BodyPartView } from "./BodyPartView";
import "../styles.css";

type ViewAngle = "front" | "back" | "lateral" | "medial";

interface BodyMapProps {
  selections: ZoneSelection[];
  onZoneSelect: (selection: ZoneSelection) => void;
  onZoneDeselect?: (zoneId: string) => void;
  onNavigate?: (layer: number, bodyPart?: string, view?: string) => void;
  multiSelect?: boolean;
  severityColors?: Record<Severity, string>;
  className?: string;
  style?: React.CSSProperties;

  // Controlled navigation (optional)
  layer?: 1 | 2;
  selectedBodyPart?: string;
  selectedView?: ViewAngle;
}

export function BodyMap({
  selections,
  onZoneSelect,
  onZoneDeselect,
  onNavigate,
  severityColors,
  className,
  style,
  layer: controlledLayer,
  selectedBodyPart: controlledBodyPart,
  selectedView: controlledView,
}: BodyMapProps) {
  const nav = useNavigation();

  // Use controlled props if provided, otherwise internal state
  const layer = controlledLayer ?? nav.layer;
  const currentBodyPart = controlledBodyPart ?? nav.selectedBodyPart;
  const currentView = controlledView ?? nav.selectedView;

  const handleBodyPartClick = (bodyPartId: string, defaultView: ViewAngle) => {
    nav.selectBodyPart(bodyPartId, defaultView);
    onNavigate?.(2, bodyPartId, defaultView);
  };

  const handleBack = () => {
    nav.goBack();
    onNavigate?.(1);
  };

  const handleZoneDeselect = (zoneId: string) => {
    onZoneDeselect?.(zoneId);
  };

  return (
    <div className={`anatomy-body-map ${className ?? ""}`} style={style}>
      {layer === 1 && (
        <FullBodyView
          onBodyPartClick={handleBodyPartClick}
          selections={selections}
        />
      )}
      {layer === 2 && currentBodyPart && currentView && (
        <BodyPartView
          bodyPartId={currentBodyPart}
          initialView={currentView}
          selections={selections}
          onZoneSelect={onZoneSelect}
          onZoneDeselect={handleZoneDeselect}
          onBack={handleBack}
          severityColors={severityColors}
        />
      )}
    </div>
  );
}
```

- [ ] **Step 2: Update components/index.ts**

Add to `packages/react/src/components/index.ts`:
```ts
export { BodyMap } from "./BodyMap";
```

- [ ] **Step 3: Create react package index.ts**

```ts
// packages/react/src/index.ts
export { BodyMap } from "./components/BodyMap";
export { SeverityPopover } from "./components/SeverityPopover";
export { FullBodyView } from "./components/FullBodyView";
export { BodyPartView } from "./components/BodyPartView";
export { ViewTabs } from "./components/ViewTabs";
export { ZoneOverlay } from "./components/ZoneOverlay";
export { useSelection } from "./hooks/useSelection";
export { useNavigation } from "./hooks/useNavigation";
```

- [ ] **Step 4: Commit**

```bash
git add packages/react/src/
git commit -m "feat(react): add BodyMap main component with 3-layer navigation"
```

---

## Task 13: Demo App

**Files:**
- Create: `apps/demo/src/main.tsx`
- Create: `apps/demo/src/App.tsx`
- Create: `apps/demo/src/App.css`
- Create: `apps/demo/tsconfig.json`

- [ ] **Step 1: Create main.tsx**

```tsx
// apps/demo/src/main.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { App } from "./App";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

- [ ] **Step 2: Create App.tsx**

```tsx
// apps/demo/src/App.tsx
import React from "react";
import { BodyMap, useSelection } from "@anatomy/react";
import type { ZoneSelection } from "@anatomy/core";
import "./App.css";

export function App() {
  const { selections, addSelection, removeSelection, clearSelections } =
    useSelection();

  const handleZoneSelect = (zone: ZoneSelection) => {
    addSelection(zone);
  };

  return (
    <div className="demo-app">
      <header className="demo-header">
        <h1>Anatomy Body Map — Demo</h1>
        <p>Click a body part to zoom in, then click zones to mark pain areas.</p>
      </header>

      <div className="demo-content">
        <div className="demo-map">
          <BodyMap
            selections={selections}
            onZoneSelect={handleZoneSelect}
            onZoneDeselect={removeSelection}
          />
        </div>

        <div className="demo-sidebar">
          <h2>Selections ({selections.length})</h2>
          {selections.length === 0 && (
            <p className="demo-empty">No zones selected yet.</p>
          )}
          {selections.map((s) => (
            <div key={s.zoneId} className={`demo-selection demo-severity-${s.severity}`}>
              <div className="demo-selection-header">
                <strong>{s.zoneName}</strong>
                <button onClick={() => removeSelection(s.zoneId)}>×</button>
              </div>
              <div className="demo-selection-meta">
                {s.bodyPart} · {s.view} · {s.severity}
              </div>
            </div>
          ))}
          {selections.length > 0 && (
            <button className="demo-clear" onClick={clearSelections}>
              Clear All
            </button>
          )}
        </div>
      </div>
    </div>
  );
}
```

- [ ] **Step 3: Create App.css**

```css
/* apps/demo/src/App.css */
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  background: #0d1117;
  color: #e0e0e0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

.demo-app {
  max-width: 1100px;
  margin: 0 auto;
  padding: 24px;
}

.demo-header {
  margin-bottom: 24px;
}

.demo-header h1 {
  font-size: 22px;
  margin-bottom: 4px;
}

.demo-header p {
  color: #888;
  font-size: 14px;
}

.demo-content {
  display: flex;
  gap: 24px;
}

.demo-map {
  flex: 1;
}

.demo-sidebar {
  width: 280px;
  flex-shrink: 0;
}

.demo-sidebar h2 {
  font-size: 16px;
  margin-bottom: 12px;
}

.demo-empty {
  color: #666;
  font-size: 13px;
}

.demo-selection {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
  padding: 10px 12px;
  margin-bottom: 8px;
  border-left: 3px solid #666;
}

.demo-severity-mild { border-left-color: #FFD600; }
.demo-severity-moderate { border-left-color: #FF9800; }
.demo-severity-severe { border-left-color: #F44336; }

.demo-selection-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.demo-selection-header button {
  background: none;
  border: none;
  color: #888;
  font-size: 18px;
  cursor: pointer;
}

.demo-selection-meta {
  font-size: 12px;
  color: #888;
  margin-top: 4px;
}

.demo-clear {
  display: block;
  width: 100%;
  margin-top: 12px;
  padding: 8px;
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 6px;
  background: transparent;
  color: #888;
  font-size: 13px;
  cursor: pointer;
}

.demo-clear:hover {
  background: rgba(255, 60, 60, 0.1);
  color: #ff5370;
}
```

- [ ] **Step 4: Create apps/demo/tsconfig.json**

```json
{
  "extends": "../../tsconfig.json",
  "include": ["src"]
}
```

- [ ] **Step 5: Build core and run demo**

```bash
cd packages/core && npx tsc
cd ../.. && npm run dev
```

Expected: Vite dev server starts. Open browser, see full body front+back placeholder with clickable regions. Click a region → Layer 2 with view tabs and placeholder SVG. (Zones won't be clickable yet since zone data is empty — that comes when real SVGs are added.)

- [ ] **Step 6: Commit**

```bash
git add apps/demo/
git commit -m "feat(demo): add demo playground app"
```

---

## Task 14: Build & Package Verification

**Files:**
- Modify: `packages/core/package.json` (add `"files"` field)
- Modify: `packages/react/package.json` (add `"files"` field)

- [ ] **Step 1: Add files field to core package.json**

Add `"files": ["dist"]` to `packages/core/package.json` so only built output is published.

- [ ] **Step 2: Add files field to react package.json**

Add `"files": ["dist"]` to `packages/react/package.json`.

- [ ] **Step 3: Build both packages**

```bash
cd packages/core && npx tsc
cd ../react && npx tsc
```

Expected: Both compile without errors. `dist/` directories created with `.js` and `.d.ts` files.

- [ ] **Step 4: Run all tests**

```bash
npm run test --workspaces --if-present
```

Expected: All tests pass across both packages.

- [ ] **Step 5: Verify demo app works end-to-end**

```bash
npm run dev
```

Open browser. Verify:
1. Layer 1: front+back body shown side by side
2. Click a body part → Layer 2 loads with back button and view tabs
3. Back button returns to Layer 1
4. Selections panel in sidebar is empty (no zones mapped yet)

- [ ] **Step 6: Commit**

```bash
git add .
git commit -m "chore: finalize build config and verify packages"
```

- [ ] **Step 7: Push to GitHub**

```bash
git push origin main
```
