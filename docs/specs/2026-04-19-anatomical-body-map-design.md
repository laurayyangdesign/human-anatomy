# Anatomical Body Map Library — Design Spec

## Context

Build a foundational, reusable anatomical body map library for clinical/therapeutic applications. This library will power future apps used by therapists (detailed view) and their clients (simplified view) for pain location mapping. The primary use cases are medical/health tracking and clinical documentation. This is a foundational library — export, persistence, condition recommendations, and patient management are left to consuming apps.

## Scope

### In Scope

- Musculoskeletal anatomy data (muscles & bones) with hierarchical taxonomy
- Interactive SVG body map with 3-layer drill-down navigation
- Two detail modes: simple (clients, fewer/larger zones per body part) and detailed (therapists, more granular zones with muscle/bone specificity)
- Severity selection popover (mild/moderate/severe) with color coding
- Multi-select across body parts
- Gender variants (male/female/neutral)
- Published as npm packages

### Out of Scope (Future)

- Internal organs, nerves, circulatory system
- Clinical keywords / condition recommendations
- PDF / intake form export
- Data persistence / storage
- 3D rendering (data model is 3D-ready)
- Patient management / authentication

## Architecture

### Monorepo — 2 Packages

```
human-anatomy/
├── packages/
│   ├── core/                    # @anatomy/core
│   │   ├── src/
│   │   │   ├── data/
│   │   │   │   ├── regions.ts        # Body region definitions (~30)
│   │   │   │   ├── muscles.ts        # Muscle groups & individual muscles
│   │   │   │   ├── bones.ts          # Bones & joints
│   │   │   │   └── index.ts
│   │   │   ├── types.ts              # TypeScript interfaces
│   │   │   ├── taxonomy.ts           # Hierarchical body part tree
│   │   │   ├── search.ts             # Lookup by name, region
│   │   │   └── index.ts
│   │   └── package.json
│   │
│   └── react/                   # @anatomy/react
│       ├── src/
│       │   ├── components/
│       │   │   ├── BodyMap.tsx        # Main component (manages 3-layer nav)
│       │   │   ├── FullBodyView.tsx   # Layer 1: front+back side by side
│       │   │   ├── BodyPartView.tsx   # Layer 2: zoomed body part with view tabs
│       │   │   ├── ZoneLayer.tsx      # Layer 3: clickable pain zones
│       │   │   ├── SeverityPopover.tsx # Mild/Moderate/Severe card popover
│       │   │   ├── SystemLayer.tsx    # Toggle muscles / bones overlay
│       │   │   └── index.ts
│       │   ├── hooks/
│       │   │   ├── useSelection.ts    # Selection state management
│       │   │   └── useNavigation.ts   # Layer navigation state
│       │   ├── svg/
│       │   │   ├── full-body-front.svg
│       │   │   ├── full-body-back.svg
│       │   │   ├── parts/             # Zoomed SVGs per body part
│       │   │   │   ├── back-front.svg
│       │   │   │   ├── back-back.svg
│       │   │   │   ├── back-lateral.svg
│       │   │   │   ├── arm-front.svg
│       │   │   │   ├── arm-back.svg
│       │   │   │   └── ... (per body part × per view)
│       │   │   └── index.ts
│       │   └── index.ts
│       └── package.json
│
├── apps/
│   └── demo/                    # Playground / demo app
├── package.json                 # Workspace root
└── tsconfig.json
```

### `@anatomy/core` — Pure TypeScript, zero dependencies

Framework-agnostic data layer. Contains all anatomical structure definitions, hierarchy, and lookup utilities.

**Exports:**

```ts
// Data access
getAllMuscles()              // () => AnatomicalStructure[]
getAllBones()                // () => AnatomicalStructure[]
getByRegion(region)         // (region: BodyRegion) => AnatomicalStructure[]
getById(id)                 // (id: string) => AnatomicalStructure | undefined
getChildren(parentId)       // (parentId: string) => AnatomicalStructure[]
search(query)               // (query: string) => AnatomicalStructure[]

// All types
type AnatomicalStructure, BodySystem, BodyRegion, BodyPart,
     BodyPartView, Zone, ZoneSelection, Severity, Annotation
```

### `@anatomy/react` — React components consuming the core

Interactive SVG body map with built-in 3-layer drill-down navigation and severity popover.

## Data Model

### Core Types

```ts
type BodySystem = "muscular" | "skeletal";

// Individual muscle or bone
interface AnatomicalStructure {
  id: string;              // "muscle.trapezius.upper"
  name: string;            // "Upper Trapezius"
  system: BodySystem;
  region: BodyRegion;
  parentId?: string;       // for hierarchy
  aliases?: string[];      // ["upper traps"]
}

// ~24 body regions
type BodyRegion =
  | "head" | "neck" | "chest" | "abdomen"
  | "upper_back" | "lower_back" | "pelvis"
  | "left_shoulder" | "right_shoulder"
  | "left_upper_arm" | "right_upper_arm"
  | "left_forearm" | "right_forearm"
  | "left_hand" | "right_hand"
  | "left_hip" | "right_hip"
  | "left_thigh" | "right_thigh"
  | "left_shin" | "right_shin"
  | "left_foot" | "right_foot";
```

### 3-Layer Navigation Types

```ts
// Layer 1: Major body parts (~10-12)
interface BodyPart {
  id: string;              // "chest", "back", "left_arm"
  name: string;            // "Chest", "Back", "Left Arm"
  side: "front" | "back" | "both";
}

// Layer 2: Views per body part
interface BodyPartView {
  bodyPartId: string;      // "back"
  view: "front" | "back" | "lateral" | "medial";
  svgPath: string;         // path to zoomed SVG
  zones: Zone[];           // mapped clickable zones
}

// Layer 3: Pain zones (pre-mapped regions in each view)
interface Zone {
  id: string;              // "back.left_scapular"
  name: string;            // "Left Scapular Zone"
  svgPathData: string;     // SVG path for this zone's shape
  relatedStructures: string[];  // muscle/bone IDs in this zone
}
```

### Severity & Selection

```ts
type Severity = "mild" | "moderate" | "severe";

// Default colors (customizable via props)
const SEVERITY_COLORS: Record<Severity, string> = {
  mild:     "#FFD600",  // yellow
  moderate: "#FF9800",  // orange
  severe:   "#F44336",  // red
};

// What the app receives when a zone is selected
interface ZoneSelection {
  zoneId: string;
  zoneName: string;
  bodyPart: string;
  view: string;
  severity: Severity;           // required
  relatedStructures: string[];
  annotation?: Annotation;      // optional notes from app
}

interface Annotation {
  label?: string;
  notes?: string;
  color?: string;
  metadata?: Record<string, any>;
}
```

## 3-Layer Drill-Down Flow

### Layer 1: Full Body Overview

- Front and back body SVGs shown **side by side**
- ~10-12 major clickable body parts (head, chest, back, abdomen, left/right arm, left/right hip, left/right leg)
- Clicking a body part navigates to Layer 2

### Layer 2: Zoomed Body Part

- Shows a zoomed-in SVG of the selected body part
- **Default view** = the side the user clicked from (e.g. clicked on front → shows front view)
- **View tabs**: Front / Back / Lateral / Medial — user can switch
- Each view has **pre-mapped clickable zones** overlaid on the SVG
- Clicking a zone triggers the severity popover (Layer 3)

### Layer 3: Severity Popover & Zone Selection

- **Card-style popover** appears near the clicked zone
- Shows the zone name and 3 severity options:
  - **Mild** — yellow circle + label
  - **Moderate** — orange circle + label
  - **Severe** — red circle + label
- User picks a severity → zone fills with that color → `onZoneSelect` fires
- **Dismiss**: clicking outside cancels (zone stays unselected)
- **Re-click selected zone**: popover reopens to change severity or remove selection

### Navigation

- **Back button / breadcrumb** to return from Layer 2 to Layer 1
- **Selections persist** — going back to Layer 1 and selecting another body part keeps previous zone selections
- **Multi-select** — user can select multiple zones across different body parts

## Component API

### Basic Usage

```tsx
import { BodyMap } from '@anatomy/react';

<BodyMap
  gender="male"
  selections={selections}
  onZoneSelect={handleZoneSelect}
/>

function handleZoneSelect(zone: ZoneSelection) {
  // { zoneId, zoneName, bodyPart, view, severity, relatedStructures }
}
```

### Full Props

| Prop | Type | Description |
|------|------|-------------|
| `mode` | `"simple" \| "detailed"` | Simple = fewer/larger zones for clients. Detailed = granular zones with muscle/bone specificity for therapists. Default: `"simple"` |
| `gender` | `"male" \| "female" \| "neutral"` | Body shape variant |
| `selections` | `ZoneSelection[]` | Controlled selection state |
| `onZoneSelect` | `(zone: ZoneSelection) => void` | Fired after severity is picked |
| `onZoneDeselect` | `(zoneId: string) => void` | Fired when zone is removed |
| `onNavigate` | `(layer, bodyPart?, view?) => void` | Optional: track navigation changes |
| `multiSelect` | `boolean` | Allow multiple zone selections (default: true) |
| `severityColors` | `Record<Severity, string>` | Override default severity colors |
| `systems` | `("muscular" \| "skeletal")[]` | Which layers to show (detailed mode) |
| `className` / `style` | `string` / `CSSProperties` | Styling overrides |

### Advanced: Controlled Navigation

```tsx
<BodyMap
  gender="female"
  layer={currentLayer}              // 1 | 2 | 3
  selectedBodyPart="back"           // jump to Layer 2
  selectedView="lateral"            // control view angle
  selections={selections}
  onZoneSelect={handleZoneSelect}
  multiSelect={true}
/>
```

## SVG Assets Required

Each body part needs zoomed SVGs for up to 4 views, with pre-mapped zone paths:

| Body Part | Views Needed |
|-----------|-------------|
| Head/Neck | front, back, lateral |
| Chest | front |
| Abdomen | front |
| Upper Back | back |
| Lower Back | back, lateral |
| Left/Right Shoulder | front, back, lateral |
| Left/Right Arm | front, back, lateral, medial |
| Left/Right Hip | front, back, lateral |
| Left/Right Leg | front, back, lateral, medial |
| Left/Right Hand | front, back |
| Left/Right Foot | front, back, lateral, medial |

Plus 2 full-body SVGs (front + back) for Layer 1.

## Verification

1. **Demo app** — run `apps/demo` to interact with all 3 layers
2. **Layer 1** — verify front+back side by side, all body parts clickable
3. **Layer 2** — verify zoomed view loads, view tabs work, zones are clickable
4. **Layer 3** — verify severity popover appears, colors apply, dismiss works
5. **Multi-select** — select zones across multiple body parts, verify all persist
6. **Re-selection** — click selected zone, change severity, verify color updates
7. **Navigation** — back button returns to Layer 1 with selections intact
8. **Core package** — import `@anatomy/core` independently, verify data access functions work
9. **Gender variants** — switch between male/female/neutral, verify SVGs change
