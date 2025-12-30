# SuperSplat Viewer Customizations

This document describes the custom modifications made to the `supersplat-viewer` package and the workflow for generating visualization apps with these features.

## New Features

### 1. Keyframe-Based Navigation (Prev/Next Controls)

The animation playback system has been redesigned to use **keyframe-based navigation** instead of continuous play/pause controls.

**Changes:**
- Removed `Play` and `Pause` buttons
- Added `Previous Keyframe` and `Next Keyframe` buttons
- Animation always starts in a **paused state**
- Users navigate through the camera animation by stepping between keyframes

**Files Modified:**
- `supersplat-viewer/src/index.html` - Replaced play/pause buttons with prev/next buttons and updated SVG icons
- `supersplat-viewer/src/ui.ts` - Updated DOM references, event listeners, and tooltips

### 2. Smooth Keyframe Transitions

When navigating between keyframes, the camera smoothly transitions instead of snapping instantly.

**Parameters (configurable in `anim-controller.ts`):**
- `transitionDuration`: 0.8 seconds (ease-in-out cubic interpolation)

**Files Modified:**
- `supersplat-viewer/src/cameras/anim-controller.ts` - Added `next()` and `prev()` methods with smooth transitions
- `supersplat-viewer/src/animation/anim-state.ts` - Added `getNextKeyframe()` and `getPrevKeyframe()` methods

### 3. Hover Effect While Paused

When the animation is paused, a subtle camera hover effect keeps the scene visually alive.

**Parameters (configurable in `anim-controller.ts`):**
```typescript
private hoverAmplitude = 0.25;      // Lateral movement strength
private hoverCycleDuration = 8;     // Seconds for one full lateral cycle
private zoomAmplitude = 0.20;       // Zoom in/out strength
private zoomCycleDuration = 12;     // Seconds for one full zoom cycle
```

To make it **slower**: increase cycle duration (e.g., `20` = 20 seconds per cycle)
To make it **stronger**: increase amplitude values

**Files Modified:**
- `supersplat-viewer/src/cameras/anim-controller.ts` - Added `applyHover()` method and hover state management
- `supersplat-viewer/src/camera-manager.ts` - Passes `isPaused` state to the animation controller

### 4. Removed Spacebar Play/Pause Shortcut

The spacebar keyboard shortcut for toggling play/pause has been removed since playback is no longer supported.

**Files Modified:**
- `supersplat-viewer/src/input-controller.ts` - Removed spacebar event handler

### 5. Always Start Paused

The viewer always starts with the animation in a paused state, regardless of URL parameters.

**Files Modified:**
- `supersplat-viewer/src/viewer.ts` - Changed `state.animationPaused = !!config.noanim` to `state.animationPaused = true`

---

## File Summary

| File | Changes |
|------|---------|
| `src/index.html` | Replaced play/pause buttons with prev/next; updated SVG icons |
| `src/ui.ts` | Updated DOM array, event listeners, tooltips for prev/next |
| `src/viewer.ts` | Always start paused |
| `src/input-controller.ts` | Removed spacebar shortcut |
| `src/animation/anim-state.ts` | Added keyframes array, `getNextKeyframe()`, `getPrevKeyframe()` |
| `src/cameras/anim-controller.ts` | Added `next()`, `prev()`, smooth transitions, hover effect |
| `src/camera-manager.ts` | Added prevKeyframe/nextKeyframe handlers, pass isPaused to controller |

---

## Build & Export Workflow

### Prerequisites
- Node.js (v20+)
- npm

### Step 1: Make Changes to supersplat-viewer

Edit source files in `supersplat-viewer/src/`:

```bash
cd /Users/ogoshi/cartoon8/supersplat-viewer
# Edit files as needed...
```

### Step 2: Build supersplat-viewer

```bash
cd /Users/ogoshi/cartoon8/supersplat-viewer
npm run build
```

This compiles:
- `src/*.ts` → `public/index.js`
- `src/index.scss` → `public/index.css`
- `src/index.html` → `public/index.html`
- `src/module/index.ts` → `dist/index.js` (module exports)

### Step 3: Rebuild supersplat

The `supersplat` project imports from `supersplat-viewer` via:
```json
"@playcanvas/supersplat-viewer": "file:../supersplat-viewer"
```

Rebuild to pick up changes:

```bash
cd /Users/ogoshi/cartoon8/supersplat
npm run build
```

### Step 4: Clear Browser Cache (Important!)

SuperSplat uses a service worker that caches the app. To see changes after a rebuild:

**Best Option:** Unregister the service worker (one-time)
1. Open DevTools (`Cmd+Option+I`)
2. Go to **Application** → **Service Workers**
3. Click **Unregister** on the supersplat worker
4. Hard refresh (`Cmd+Shift+R`)

**Alternative:** Clear all site data
1. Go to **Application** → **Storage**
2. Click **"Clear site data"**

**Fallback:** Use a different port to bypass cache entirely
```bash
npx serve dist -C -p 3002
```

### Step 5: Export Visualization

1. Open SuperSplat at `http://localhost:3001`
2. Load your `.ply` file
3. Set up camera animation keyframes
4. **File → Export → Viewer**
5. Save to desired location

### Step 6: Test Exported Visualization

```bash
cd /path/to/exported/folder
python3 -m http.server 8090
# Open http://localhost:8090
```

---

## Quick Reference Commands

```bash
# Full rebuild workflow (single command)
cd /Users/ogoshi/cartoon8 && ./build-all.sh

# Serve SuperSplat editor (fresh port to bypass cache)
cd /Users/ogoshi/cartoon8 && ./serve.sh

# Or manually:
cd /Users/ogoshi/cartoon8/supersplat && npx serve dist -C -p 3001

# Test exported visualization
cd /path/to/export && python3 -m http.server 8090
```

---

## Customization Tips

### Adjust Hover Effect
Edit `supersplat-viewer/src/cameras/anim-controller.ts`:
```typescript
private hoverAmplitude = 0.06;  // Increase for more movement
private hoverFrequency = 0.2;   // Increase for faster oscillation
```

### Adjust Transition Speed
Edit `supersplat-viewer/src/cameras/anim-controller.ts`:
```typescript
private transitionDuration = 0.8;  // Seconds (lower = faster)
```

### Change Easing Function
The transition uses ease-in-out cubic. Modify in `anim-controller.ts`:
```typescript
const ease = t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2;
```

---

## Troubleshooting

### Export still has old play/pause buttons
1. Ensure `supersplat-viewer` is rebuilt: `npm run build`
2. Ensure `supersplat` is rebuilt: `npm run build`
3. Clear browser cache or use a different port
4. Export a **new** visualization (existing exports won't change)

### Buttons don't appear
- Verify your `.ply` file has animation data in `settings.json`
- Prev/Next buttons only show when `state.hasAnimation` is true

### Hover effect too strong/weak
- Adjust `hoverAmplitude` in `anim-controller.ts`
- Rebuild both projects after changes
