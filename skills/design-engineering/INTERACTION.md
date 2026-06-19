# Interaction and motion tooling reference

Load this when motion or interaction outgrows CSS and a library is the right call: orchestrated React motion, scrubbed scroll choreography, interactive illustration, 3D, generative or shader work. `MOTION.md` is the native floor and the first thing to try; this is the layer above it, with full recipes per tool.

The judgment that governs all of it: **reach for the lightest tool that does the job, and earn every kilobyte and every frame.** A 200kb 3D hero that adds a second to LCP for decoration is bad work no matter how good the scene looks. Native first, then this.

---

## Pick the tool by the job

| The job | Tool | Why not the lighter option |
|---|---|---|
| State, enter/exit, hover, simple transitions | CSS + platform-native (`MOTION.md`) | nothing here needed |
| Orchestrated React motion: layout shifts, shared-element, exit animation, gesture + spring, staggered sequences | **Motion** (formerly Framer Motion) | CSS cannot animate layout changes or coordinate exit cleanly in React |
| Timeline-precise or scroll-scrubbed sequences, pinning, cross-element choreography, non-React | **GSAP + ScrollTrigger** | CSS scroll-timeline covers single simple cases, not scrubbed multi-step timelines |
| Interactive, state-driven illustration: reactive mascots, animated icons, onboarding that responds to input/data | **Rive** | Lottie is linear playback only; it cannot react to state |
| Linear vector animation handed off from After Effects | **Lottie** (`.lottie`) | use Rive instead the moment it needs to be interactive |
| Real 3D: product viewers, spatial scenes, depth that carries meaning | **Three.js / React Three Fiber + drei** | a video or image if the 3D is purely decorative |
| Generative fields, particles, audio-reactive, creative coding | **p5.js** to prototype, raw Canvas/WebGL to ship | p5 is ~900kb; do not ship it unexamined |
| Custom visual effects: noise, dissolves, gradient meshes, displacement | **GLSL shaders** (via R3F or a fragment-shader canvas) | CSS filters strain and break past a point |

---

## The obligations that apply to every tool below

These are non-negotiable regardless of which library you reach for. They are what make this senior work rather than a demo.

**Budget the weight, and lazy-load it.** Know the cost before you add it.

| Library | Approx min gzipped | Note |
|---|---|---|
| Motion (`m` + `LazyMotion`) | ~5 to 6kb | full `motion` is ~34kb; use `LazyMotion` to defer features |
| GSAP core | ~23kb | ScrollTrigger adds ~12kb; free including all plugins since 2025 |
| Rive (`react-canvas`) | ~80kb wasm + runtime | one runtime serves any number of `.riv` files |
| Lottie (`dotlottie-react`) | ~40 to 60kb | `.lottie` is far smaller than raw `.json` |
| Three.js | ~150kb+ | tree-shake; import from `three/examples` selectively |
| p5.js | ~900kb full | the reason to port sketches to raw Canvas for production |

None of these may block first paint. Code-split the component, lazy-load it below the fold or on interaction, and never put a WebGL or p5 hero in the critical path of LCP.

**Canvas and WebGL are invisible to assistive tech.** A `<canvas>` is an opaque pixel buffer to a screen reader. Anything essential rendered inside it must also exist as real DOM. Mark purely decorative canvases `aria-hidden="true"`, never trap keyboard focus inside a 3D scene, and provide a text equivalent for anything that conveys meaning.

**Reduced motion gates all of it.** `prefers-reduced-motion: reduce` is not only for CSS. A Rive state machine, a GSAP timeline, an `useFrame` loop, a p5 draw loop, and a shader's time uniform all must check it and freeze, pause, or fall back to a static frame. Author the full motion, then provide the calm path.

**Always ship a fallback.** A static poster frame behind every heavy scene, an SSR-safe path, and a working experience on an old GPU or with JS disabled. Heavy interactive layers are enhancement, not foundation.

**The gimmick test.** Before adding 3D, generative, or shader work, answer in one line what it communicates. If the honest answer is "it looks sophisticated," cut it. Restraint applies hardest exactly where the tool is most impressive.

**Verify by watching, even more here.** A state machine, a physics drag, or a scrubbed timeline cannot be reviewed in a diff. Run it, watch it at full and at quarter speed, test the interrupt, test it on a real phone. "The config looks right" is not the stop signal.

---

## Motion (Framer Motion)

The library rebranded to Motion; import from `motion/react`. Use it where its value actually is: layout and shared-element transitions, exit animation, gesture-plus-spring, and orchestrated sequences. For a one-off hover or fade, stay in CSS.

**Keep the bundle small with `LazyMotion`.** The plain `motion` component bundles every feature. Swap to the `m` component and load features lazily to drop from ~34kb to ~6kb:

```jsx
import { LazyMotion, domAnimation, m } from "motion/react";

<LazyMotion features={domAnimation}>
  <m.div animate={{ opacity: 1 }} />
</LazyMotion>
// domMax adds drag and layout; load it only where you use them
```

**Exit animation needs `AnimatePresence`.** React unmounts immediately, so an exit transition has nowhere to run without it. `initial={false}` suppresses the mount animation on first render; `mode="popLayout"` lets leaving items animate out while the rest reflow.

```jsx
<AnimatePresence initial={false} mode="popLayout">
  {items.map((item) => (
    <m.li
      key={item.id}
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -8 }}
      transition={{ duration: 0.2, ease: [0.23, 1, 0.32, 1] }}
    />
  ))}
</AnimatePresence>
```

**Shared-element transitions with `layoutId`.** Two elements with the same `layoutId` morph into each other across mount/unmount: a thumbnail growing into a hero, a tab indicator sliding. This was the headline reason to use the library; note that the View Transitions API (`MOTION.md`) now does this natively for many cases, so check whether you need the dependency.

```jsx
{selected ? (
  <m.div layoutId="card" />          // morphs from the grid item to here
) : (
  items.map((i) => <m.div layoutId={i === active ? "card" : undefined} />)
)}
```

**Orchestrate with variants.** Parent variants drive children through propagation, so a list staggers without per-item delays:

```jsx
const list = { show: { transition: { staggerChildren: 0.05, delayChildren: 0.1 } } };
const item = { hidden: { opacity: 0, y: 12 }, show: { opacity: 1, y: 0 } };

<m.ul variants={list} initial="hidden" animate="show">
  {data.map((d) => <m.li key={d.id} variants={item} />)}
</m.ul>
```

**Gesture with momentum.** `drag` plus an elastic boundary and a velocity-aware end handler gives a physical flick-to-dismiss, the pattern from `MOTION.md` made concrete:

```jsx
const x = useMotionValue(0);
const opacity = useTransform(x, [-200, 0, 200], [0, 1, 0]);

<m.div
  style={{ x, opacity }}
  drag="x"
  dragConstraints={{ left: 0, right: 0 }}
  dragElastic={0.5}
  onDragEnd={(e, info) => {
    if (Math.abs(info.velocity.x) > 500 || Math.abs(info.offset.x) > 120) onDismiss();
  }}
/>
```

**Scroll-linked values.** `useScroll` plus `useSpring` gives a smoothed progress indicator without a scroll listener:

```jsx
const { scrollYProgress } = useScroll();
const scaleX = useSpring(scrollYProgress, { stiffness: 120, damping: 30 });
<m.div style={{ scaleX, transformOrigin: "left" }} className="progress-bar" />
```

**The hardware-acceleration caveat (from `MOTION.md`).** The shorthand `x` / `y` / `scale` props animate on the main thread, so they drop frames when the page is busy. Where a transform is independent and the scene is heavy, animate the full `transform` string to stay on the compositor, or prefer a CSS transition for that piece.

**Respect reduced motion.** `useReducedMotion()` returns the user's setting; collapse positional motion to a plain opacity change when true.

```jsx
const reduce = useReducedMotion();
<m.div animate={reduce ? { opacity: 1 } : { opacity: 1, y: 0 }} />
```

---

## GSAP

Reach for GSAP when motion needs a precise timeline, scroll scrubbing, pinning, or coordination across many elements, and for non-React contexts. As of the 2025 Webflow acquisition the whole library including ScrollTrigger, SplitText, and Flip is free.

**Timelines sequence cleanly.** A timeline chains tweens with a position parameter, so you compose a sequence instead of juggling delays:

```js
import { gsap } from "gsap";

const tl = gsap.timeline({ defaults: { ease: "power3.out", duration: 0.6 } });
tl.from(".title", { y: 40, opacity: 0 })
  .from(".subtitle", { y: 20, opacity: 0 }, "-=0.3")   // overlap previous by 0.3s
  .from(".cta", { scale: 0.9, opacity: 0 }, "<");        // start with previous
```

**ScrollTrigger for scroll choreography.** `scrub` ties progress to scroll position; `pin` holds the element while the sequence plays. Keep `markers: true` on while building, remove before ship.

```js
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);

gsap.to(".panel", {
  xPercent: -100 * (panels - 1),
  ease: "none",
  scrollTrigger: {
    trigger: ".container",
    pin: true,
    scrub: 1,            // 1 = smooth catch-up; true = locked to scroll
    end: () => "+=" + document.querySelector(".container").offsetWidth,
  },
});
```

**Clean up in React with `useGSAP`.** The `@gsap/react` hook scopes selectors and reverts every animation and ScrollTrigger on unmount, which is the usual source of leaks and double-binding.

```jsx
import { useGSAP } from "@gsap/react";

const container = useRef();
useGSAP(() => {
  gsap.from(".item", { y: 30, opacity: 0, stagger: 0.08 });
}, { scope: container });   // selectors resolve inside container, auto-cleanup

return <div ref={container}>{/* .item children */}</div>;
```

**Responsive and reduced motion with `matchMedia`.** Register motion per media query; GSAP reverts a context automatically when its query stops matching, so the reduced-motion branch is a first-class path, not an afterthought.

```js
const mm = gsap.matchMedia();
mm.add(
  { reduce: "(prefers-reduced-motion: reduce)", ok: "(prefers-reduced-motion: no-preference)" },
  (ctx) => {
    const { reduce } = ctx.conditions;
    gsap.from(".hero", reduce ? { opacity: 0 } : { opacity: 0, y: 60, duration: 0.8 });
  }
);
```

**SplitText and Flip** cover the two jobs people drop to GSAP for most: animating text by character/word/line, and morphing between two DOM states (Flip records positions, you change the DOM, Flip tweens the difference, the same idea as View Transitions but with full control).

---

## Rive over Lottie for anything interactive

Rive animations carry a **state machine**: named states, transitions, and inputs the runtime can set, so the animation reacts to hover, click, scroll, data, or progress. Lottie is linear playback exported from After Effects; you can play segments but it cannot respond to state. Choose Rive for reactive icons, mascots, onboarding, and progress; choose Lottie only when a designer hands off a linear clip you just need to play.

**Wire a state machine input in React.** Get the input by name from the machine, then set it from app state. Boolean and number inputs hold a value; trigger inputs fire once.

```jsx
import { useRive, useStateMachineInput } from "@rive-app/react-canvas";

const { rive, RiveComponent } = useRive({
  src: "/character.riv",
  stateMachines: "State Machine 1",
  autoplay: true,
});

const hover = useStateMachineInput(rive, "State Machine 1", "hover");   // boolean
const level = useStateMachineInput(rive, "State Machine 1", "level");   // number

return (
  <RiveComponent
    style={{ width: 240, height: 240 }}
    onMouseEnter={() => hover && (hover.value = true)}
    onMouseLeave={() => hover && (hover.value = false)}
  />
);
// drive from data: useEffect(() => { if (level) level.value = score; }, [score]);
```

**Data binding (2024+)** lets a `.riv` read view-model properties directly, so a single file can show live text, colors, and numbers without re-exporting. Prefer it over swapping files when the animation reflects changing data.

**Performance and fallback.** One Rive runtime serves many `.riv` files, and the files are tiny because they are vectors plus a state graph rather than baked frames. Use the `react-canvas` runtime for most work and the WebGL2 runtime only when you need meshes or heavy raster. Honor reduced motion by pausing and showing a resting frame:

```jsx
const reduce = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
useEffect(() => { if (reduce && rive) rive.pause(); }, [reduce, rive]);
```

Provide an `aria-label` on the wrapper since the canvas itself announces nothing.

---

## Three.js with React Three Fiber

Use real 3D when depth carries meaning: a product you rotate, a configurator, a scene the user moves through. R3F expresses a Three scene as a component tree; drei supplies the helpers you would otherwise rewrite.

**Scene skeleton, client-only.** The canvas cannot server-render, so in Next dynamically import it with `ssr: false` and wrap loaders in Suspense. Cap device pixel ratio so retina phones do not render at 3x and overheat.

```jsx
import { Canvas, useFrame } from "@react-three/fiber";
import { OrbitControls, Environment, useGLTF } from "@react-three/drei";

function Model() {
  const { scene } = useGLTF("/model-draco.glb");   // draco-compressed
  const ref = useRef();
  useFrame((_, delta) => { ref.current.rotation.y += delta * 0.2; });
  return <primitive ref={ref} object={scene} />;
}
useGLTF.preload("/model-draco.glb");

<Canvas dpr={[1, 2]} camera={{ position: [0, 0, 5], fov: 45 }} frameloop="demand">
  <ambientLight intensity={0.4} />
  <directionalLight position={[5, 5, 5]} />
  <Suspense fallback={null}><Model /><Environment preset="city" /></Suspense>
  <OrbitControls enablePan={false} />
</Canvas>
```

**Render on demand for anything not continuously moving.** `frameloop="demand"` stops the render loop until something changes; call `invalidate()` after a state change. A static model viewer that renders every frame drains battery for nothing.

**The performance budget is the design.**

- **Draw calls.** Each unique mesh-material pair is a draw call. Repeated objects (a field of the same shape) must use `<Instances>` / instanced meshes, turning thousands of draw calls into one.
- **Geometry and textures.** Compress meshes with Draco or meshopt, keep textures at the size they display (a 4k texture on a 200px element is waste), and use KTX2 compressed textures for GPU memory.
- **Adapt to the device.** drei's `<PerformanceMonitor>` plus `<AdaptiveDpr>` drop resolution when the frame rate falls, so a weak GPU stays smooth instead of janking.
- **Dispose.** Free geometries, materials, and textures on unmount; R3F disposes its own tree but imported assets and manual objects leak otherwise.

**Postprocessing is expensive.** Effects from `@react-three/postprocessing` (bloom, depth of field) run full-screen passes. Add them deliberately, test on a mid phone, and drop them under load.

**Accessibility and fallback.** Render a poster image first and swap in the canvas after load, freeze `useFrame` under reduced motion, keep all controls and content reachable in DOM, and never gate essential information behind the 3D.

---

## p5.js, Canvas, and generative work

p5 is the fastest way to sketch generative and creative-coding ideas, but at ~900kb it is a prototyping tool. For production, port the sketch to raw Canvas 2D or WebGL. Whichever you ship, the loop discipline is the same.

**Use p5 in instance mode** so it does not grab globals and can mount into a React ref:

```jsx
import p5 from "p5";
useEffect(() => {
  const sketch = (p) => {
    p.setup = () => { p.createCanvas(p.windowWidth, p.windowHeight); };
    p.draw = () => { /* per-frame */ };
  };
  const instance = new p5(sketch, ref.current);
  return () => instance.remove();   // critical cleanup
}, []);
```

**A production-grade Canvas loop** scales for device pixel ratio so it is crisp, caps the work, pauses when the tab is hidden, and respects reduced motion:

```js
const dpr = Math.min(window.devicePixelRatio, 2);
canvas.width = w * dpr; canvas.height = h * dpr;
ctx.scale(dpr, dpr);                       // crisp on retina, capped at 2x

const reduce = matchMedia("(prefers-reduced-motion: reduce)").matches;
const COUNT = reduce ? 0 : Math.min(180, Math.floor(w / 8));   // scale to viewport

let raf;
const loop = () => { update(); render(); raf = requestAnimationFrame(loop); };
document.addEventListener("visibilitychange", () => {
  document.hidden ? cancelAnimationFrame(raf) : (raf = requestAnimationFrame(loop));
});
if (!reduce) loop(); else renderStaticFrame();
```

**Audio-reactive fields** read a Web Audio `AnalyserNode`: connect a source, pull `getByteFrequencyData` each frame, and drive size, color, or motion from the bands. Smooth the values (a low-pass on each bin) so the visual breathes rather than strobes.

**Accessibility.** A canvas is opaque to assistive tech, so set `aria-hidden="true"` on a purely decorative one and make sure nothing essential lives only in pixels. Keep particle counts honest on mobile; a field that is gorgeous on a laptop can melt a phone.

---

## GLSL shaders

When a custom visual effect (noise field, dissolve, gradient mesh, displacement) outruns CSS filters, write a fragment shader. It runs per pixel on the GPU, so it is cheap for full-screen effects that would thrash on the main thread, and expensive if the per-pixel math is heavy.

**A shader material in R3F.** drei's `shaderMaterial` builds the material from uniforms and the two shader strings; drive `uTime` from `useFrame`, and freeze it under reduced motion so the surface goes still.

```jsx
import { shaderMaterial } from "@react-three/drei";
import { extend, useFrame } from "@react-three/fiber";

const GradientMaterial = shaderMaterial(
  { uTime: 0, uColorA: new THREE.Color("#5b21b6"), uColorB: new THREE.Color("#0ea5e9") },
  /* vertex */   `varying vec2 vUv; void main(){ vUv = uv; gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);} `,
  /* fragment */ `uniform float uTime; uniform vec3 uColorA, uColorB; varying vec2 vUv;
    void main(){ float t = 0.5 + 0.5*sin(uTime + vUv.x*3.0);
                 gl_FragColor = vec4(mix(uColorA, uColorB, t), 1.0); }`
);
extend({ GradientMaterial });

function Plane() {
  const ref = useRef();
  const reduce = useReducedMotion();
  useFrame((_, dt) => { if (!reduce) ref.current.uTime += dt; });
  return <mesh><planeGeometry args={[2, 2]} /><gradientMaterial ref={ref} /></mesh>;
}
```

**Performance.** Fragment shaders execute once per pixel per frame, so cost scales with screen area: avoid heavy loops and dynamic branching, set `precision mediump float` on mobile, and keep texture lookups few. Pass `uResolution` and `uMouse` as uniforms rather than recompiling. For a DOM-only effect with no Three scene, the same fragment shader runs in a small full-screen WebGL quad.

---

## When to walk it back

The most senior move in this whole file is removing the library. If a CSS transition does the job, the Motion dependency is debt. If a poster image communicates the product, the Three scene is weight the user pays for and the screen reader cannot see. Reach for these tools when the interaction genuinely needs them, ship them with the budget and the fallback and the reduced-motion path intact, and cut them the moment a lighter answer appears.
