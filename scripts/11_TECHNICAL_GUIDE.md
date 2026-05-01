# ADVANCED EXPERIENCE GUIDE
## Cross-Brand Technical & Creative Reference
### For Senior Developers + Creative Directors

---

## 1. UNIVERSAL SCROLL ARCHITECTURE

All 10 experiences share the same foundational scroll architecture. This is the core infrastructure:

```
[Browser Scroll Event]
        ↓
[ScrollTrigger (GSAP)]
        ↓
[scrollProgress: 0–1 float]
        ↓
[Scene Router → Active Scene Component]
        ↓
[Scene-specific animation system]
        ↓
[Three.js Renderer → Canvas]
```

### The ScrollProgress Contract

Every animation, camera position, and effect is derived from a single `scrollProgress` value (0–1):

```javascript
// Global scroll progress store
const useScrollProgress = create((set) => ({
  progress: 0,
  velocity: 0,
  setProgress: (p, v) => set({ progress: p, velocity: v }),
}));

// ScrollTrigger setup
gsap.registerPlugin(ScrollTrigger);
ScrollTrigger.create({
  trigger: '#root',
  start: 'top top',
  end: 'bottom bottom',
  onUpdate: (self) => {
    setProgress(self.progress, self.getVelocity() / 1000);
  },
  scrub: true,
});
```

### Scene Routing Logic

```javascript
const SCENES = [
  { name: 'scene1', start: 0.00, end: 0.15 },
  { name: 'scene2', start: 0.15, end: 0.30 },
  // ... etc
];

const activeScene = SCENES.find(s => 
  scrollProgress >= s.start && scrollProgress < s.end
);

// Local scene progress (0–1 within scene)
const sceneProgress = (scrollProgress - activeScene.start) 
  / (activeScene.end - activeScene.start);
```

---

## 2. CAMERA RIG SYSTEM (Universal)

All 10 experiences use the same camera rig, driven by keyframes on a CatmullRom spline:

```javascript
// Camera keyframe definition
const cameraKeyframes = [
  { t: 0.00, position: [0, 10, 20], lookAt: [0, 0, 0], fov: 60 },
  { t: 0.15, position: [0, 2, 5],  lookAt: [0, 1, 0], fov: 35 },
  { t: 0.30, position: [-3, 1, 8], lookAt: [0, 0, 0], fov: 50 },
  // ... etc
];

// Spline interpolation
const cameraPath = new THREE.CatmullRomCurve3(
  keyframes.map(k => new THREE.Vector3(...k.position))
);

// Per-frame update
useFrame(() => {
  const t = scrollProgress;
  const pos = cameraPath.getPoint(t);
  const lookAt = lookAtPath.getPoint(Math.min(t + 0.01, 1));
  
  // Smooth camera movement
  camera.position.lerp(pos, 0.05);
  camera.lookAt(lookAt);
  
  // FOV interpolation
  const fov = interpolateFov(keyframes, t);
  camera.fov = THREE.MathUtils.lerp(camera.fov, fov, 0.05);
  camera.updateProjectionMatrix();
});
```

---

## 3. SCROLL VELOCITY EFFECTS (Cross-Brand)

Every experience responds to scroll velocity differently. Here's the universal system:

```javascript
const useScrollVelocity = () => {
  const lastProgress = useRef(0);
  const velocity = useRef(0);
  
  useFrame(() => {
    const current = scrollProgress.getState().progress;
    const delta = current - lastProgress.current;
    velocity.current = THREE.MathUtils.lerp(velocity.current, delta * 60, 0.1);
    lastProgress.current = current;
  });
  
  return velocity;
};

// Usage in shaders
material.uniforms.uVelocity.value = velocity.current;
```

### Velocity Effect Matrix

| Effect | Low Velocity | High Velocity |
|--------|-------------|---------------|
| Particle systems | Slow drift | Explosive scatter |
| Camera | Smooth float | Aggressive snap |
| Transitions | Gradual dissolve | Hard cut |
| Motion blur | None | Maximum |
| Sound | Ambient | Beat-driven |
| Chromatic aberration | 0 | 0.05 |

---

## 4. MANDATORY PAUSE SYSTEM

Used in: Jordan (Scene 1), Meek Mill (Scene 4), Kanye (Scene 4), Rawme Hood (Scene 6)

```javascript
const useForcedPause = (triggerProgress, duration) => {
  const isPaused = useRef(false);
  
  useEffect(() => {
    const unsubscribe = useScrollProgress.subscribe(({ progress }) => {
      if (Math.abs(progress - triggerProgress) < 0.005 && !isPaused.current) {
        isPaused.current = true;
        
        // Lock scroll
        const scrollTop = window.scrollY;
        window.addEventListener('scroll', () => {
          window.scrollTo(0, scrollTop);
        });
        
        // Unlock after duration
        setTimeout(() => {
          isPaused.current = false;
          window.removeEventListener('scroll', lockHandler);
        }, duration);
      }
    });
    return unsubscribe;
  }, [triggerProgress, duration]);
};
```

---

## 5. PARTICLE SYSTEM CLASSES (Reusable Across Brands)

### Class A: Atmospheric (Adidas Scene 2, Karan Aujla Scene 2)
- Count: 10,000–50,000
- Behavior: Wind-driven drift, gravity-affected
- Shader: Simple points with velocity-based elongation
- Use case: Dust, pollen, snowflakes, crowd light

### Class B: Explosive (Jordan Scene 4, Meek Mill Scene 4)  
- Count: 100,000–1,000,000
- Behavior: Outward impulse + gravity + friction
- Shader: Tapered line segments (speed = length)
- Use case: Explosions, shatters, crowd bursts

### Class C: Formation (Jordan Scene 6, Meek Mill Scene 6)
- Count: 200,000–500,000
- Behavior: Spring-driven toward target positions
- Shader: Circular points with emissive core
- Use case: Logo formation, bird flock, crowd shape

### Class D: GPGPU Large-Scale (Scout Scene 4, Rawme Hood Scene 1)
- Count: 1,000,000–2,000,000
- Behavior: GPU-computed, complex rules
- Shader: GPU simulation on FBO textures
- Use case: Static noise, massive crowds, cosmic scale

```javascript
// Base particle system component
const ParticleSystem = ({ count, type, target, velocity }) => {
  const mesh = useRef();
  const [positions, setPositions] = useState(() => 
    new Float32Array(count * 3).map(() => (Math.random() - 0.5) * 10)
  );
  
  const { progress } = useScrollProgress();
  
  useFrame((state, delta) => {
    // Type-specific update logic
    switch(type) {
      case 'formation':
        updateFormation(positions, target, progress, delta);
        break;
      case 'explosive':
        updateExplosive(positions, velocity, delta);
        break;
    }
    mesh.current.geometry.attributes.position.needsUpdate = true;
  });
};
```

---

## 6. MATERIAL LIBRARY (Shared Shaders)

### Emissive Brand Materials

Each brand has a "signature material" — shared properties with unique color:

| Brand | Primary Color | Secondary | Roughness | Metalness |
|-------|--------------|-----------|-----------|-----------|
| Adidas | #FFFFFF | #000000 | 0.8 | 0.0 |
| Puma | #FF6B35 | #1A1A2E | 0.6 | 0.1 |
| Jordan | #C8102E | #000000 | 0.4 | 0.2 |
| Scout | #FF6B00 | #0A0A0A | 0.3 | 0.0 |
| Jonathan | #00D4FF | #000814 | 0.2 | 0.3 |
| Samay | #00FF41 | #0D0D0D | 0.9 | 0.0 |
| Karan Aujla | #D4AF37 | #1A0A00 | 0.5 | 0.7 |
| Meek Mill | #004C54 | #000000 | 0.7 | 0.1 |
| Kanye | #C9B99A | #0A0A0A | 0.9 | 0.0 |
| Rawme Hood | #00FF41 | #0D0D0D | 1.0 | 0.0 |

### Universal Custom Shader Uniforms

Every experience injects these into all custom shaders:

```glsl
uniform float uScrollProgress;  // 0–1 global
uniform float uSceneProgress;   // 0–1 within scene  
uniform float uVelocity;        // scroll velocity
uniform float uTime;            // elapsed time (seconds)
uniform vec2  uMouse;           // normalized mouse position
uniform float uMobileAspect;    // 1.0 desktop, varies mobile
```

---

## 7. SOUND DESIGN ARCHITECTURE

### Audio Layer System (All Brands)

```
Layer 1: AMBIENT (always on, volume responds to scene)
Layer 2: MUSIC (main theme, scene-variant)
Layer 3: INTERACTION (triggered by user actions)
Layer 4: CINEMATIC (triggered by scroll milestones)
Layer 5: HAPTIC (device vibration on mobile for key moments)
```

```javascript
// Howler.js multi-layer audio system
const AudioSystem = {
  ambient: new Howl({ src: ['ambient.mp3'], loop: true, volume: 0.3 }),
  music: new Howl({ src: ['theme.mp3'], loop: true, volume: 0 }),
  
  setScene: (sceneIndex) => {
    // Crossfade between scene audio variants
    ambient.fade(ambient.volume(), sceneVolumes[sceneIndex].ambient, 1000);
    music.fade(music.volume(), sceneVolumes[sceneIndex].music, 1000);
  },
  
  trigger: (sound) => {
    new Howl({ src: [`sfx/${sound}.mp3`] }).play();
  }
};
```

### Scroll-Reactive Music

Jonathan Gaming and Kanye experiences use scroll-reactive audio:

```javascript
// Map scroll velocity to BPM
const reactivePlayback = () => {
  const { velocity } = useScrollProgress.getState();
  const playbackRate = THREE.MathUtils.clamp(
    1.0 + velocity * 0.5, // base + velocity contribution
    0.5, // minimum (slow scroll = slow music)
    2.0  // maximum (fast scroll = double speed)
  );
  music.rate(playbackRate);
};
```

---

## 8. POST-PROCESSING PIPELINE (Universal)

All experiences use the same post-processing pipeline, with per-scene configuration:

```javascript
// @react-three/postprocessing
<EffectComposer>
  <Bloom 
    intensity={sceneConfig.bloom.intensity}
    luminanceThreshold={sceneConfig.bloom.threshold}
  />
  <DepthOfField
    focusDistance={sceneConfig.dof.focusDistance}
    focalLength={sceneConfig.dof.focalLength}
    bokehScale={sceneConfig.dof.bokehScale}
  />
  <ChromaticAberration
    offset={[sceneConfig.ca.x, sceneConfig.ca.y]}
  />
  <FilmGrain
    premultiply
    blendFunction={BlendFunction.SOFT_LIGHT}
    opacity={sceneConfig.grain}
  />
  <Vignette
    darkness={sceneConfig.vignette}
  />
  {sceneConfig.motionBlur && (
    <MotionBlur />
  )}
</EffectComposer>
```

### Per-Brand Post-Processing Default

| Brand | Bloom | Grain | Vignette | CA | DOF |
|-------|-------|-------|----------|----|-----|
| Adidas | High | Low | Medium | Low | Medium |
| Puma | Medium | High | High | Medium | High |
| Jordan | High | None | Low | None | High |
| Scout | Medium | None | Medium | None | Low |
| Jonathan | Very High | None | Low | None | Medium |
| Samay | None | High | None | High | None |
| Karan Aujla | High | Medium | High | Low | High |
| Meek Mill | Low | High | High | None | Medium |
| Kanye | Medium | Low | Medium | None | Medium |
| Rawme Hood | None | Very High | High | Very High | None |

---

## 9. MOBILE OPTIMIZATION STRATEGY

All experiences must work on mobile (the primary device for Indian and global youth audiences).

### Adaptive Quality System

```javascript
const useQualityPreset = () => {
  const [preset, setPreset] = useState('auto');
  
  useEffect(() => {
    const cores = navigator.hardwareConcurrency || 2;
    const memory = navigator.deviceMemory || 2;
    const isHighEnd = cores >= 6 && memory >= 4;
    const isMidRange = cores >= 4 && memory >= 2;
    
    setPreset(isHighEnd ? 'high' : isMidRange ? 'medium' : 'low');
  }, []);
  
  return QUALITY_PRESETS[preset];
};

const QUALITY_PRESETS = {
  high:   { particles: 1.0,  shadows: true,  postFX: true,  resolution: 1.0 },
  medium: { particles: 0.5,  shadows: false, postFX: true,  resolution: 0.75 },
  low:    { particles: 0.25, shadows: false, postFX: false, resolution: 0.5 },
};
```

### Mobile-Specific Experience Adaptations

| Scene Type | Desktop | Mobile |
|-----------|---------|--------|
| Particle count | 500k+ | 100k max |
| Post-processing | Full stack | Bloom only |
| Physics | Real-time | Baked animation |
| Video textures | Full HD | 480p |
| Shadow maps | 2048px | Disabled |
| GPGPU | Yes | No (fallback: CPU) |

---

## 10. PERFORMANCE BENCHMARKS

Target benchmarks before delivery:

| Metric | Target | Minimum |
|--------|--------|---------|
| FPS (Desktop) | 60fps | 45fps |
| FPS (Mobile) | 30fps | 24fps |
| Initial Load | <3s | <5s |
| Time to Interactive | <5s | <8s |
| Memory Usage | <512MB | <1GB |
| GPU Memory | <256MB | <512MB |

### Performance Testing Tools

```bash
# Chrome DevTools Performance tab
# GPU profiling: chrome://tracing with "GPU" category

# Three.js stats
import Stats from 'three/examples/jsm/libs/stats.module';
const stats = new Stats();
document.body.appendChild(stats.dom);
// Inspect: FPS, Memory, Render calls

# spector.js for GPU debugging
npm install spector.js
```

---

## 11. ACCESSIBILITY REQUIREMENTS

Even premium experiences must be accessible:

1. **Reduced Motion:** `prefers-reduced-motion: reduce` → static version (no scroll animations)
2. **Keyboard Navigation:** All CTAs reachable via Tab
3. **Screen Reader:** Semantic `aria-label` on all interactive elements
4. **Color Contrast:** CTAs pass WCAG AA (4.5:1 minimum)
5. **Loading States:** Visible progress indicator while 3D assets load

```javascript
const prefersReducedMotion = window.matchMedia(
  '(prefers-reduced-motion: reduce)'
).matches;

if (prefersReducedMotion) {
  // Render static hero image instead of 3D
  return <StaticHero />;
}
```

---

## 12. DEPLOYMENT CONSIDERATIONS

### Hosting Requirements

- **CDN:** All 3D assets (GLTF, textures, HDRI) on Cloudflare R2 or AWS S3 + CloudFront
- **GLTF Compression:** Draco geometry + KTX2 textures (70–80% size reduction)
- **Progressive Loading:** `Suspense` boundaries with placeholder geometry

```javascript
// Asset loading strategy
const HeavyScene = React.lazy(() => import('./scenes/HeavyScene'));

<Suspense fallback={<LowPolyPlaceholder />}>
  <HeavyScene />
</Suspense>
```

### Asset Budget per Brand

| Asset Type | Budget |
|-----------|--------|
| GLTF models (total) | <15MB |
| Textures (total) | <25MB |
| Audio (total) | <10MB |
| HDRI environments | <5MB |
| Total bundle | <60MB |

---

*This technical guide is maintained alongside the cinematic scripts.*  
*Last updated: May 2026*  
*For: Premium portfolio use — ₹5L–₹10L client tier*
