# Notes on web game VFX

Personal working notes on particle effects, runtime performance and authoring
workflow for HTML5 games.

Published at <https://sonaarajyan03.github.io/web-vfx-notes/>

These are personal notes, not documentation for any product.

## Weather that has to run in both PixiJS and Three.js: five mistakes I made

I exported one snow and rain effect from NixieFX and loaded it in a PixiJS 8 scene and a Three.js r185 scene with the same seed. The particle simulation is shared, so positions matched between the two to about 1e-15 world units. Everything that went wrong was in how I set up or judged the drawing.

1. **No projection in Pixi.** Without one, `PixiVfxRenderer` draws 1 world unit as 1 pixel and flips y. My 8-unit-wide snow box became an 8 × 4 px patch off the top-left corner, and 0 of 280 flakes were on the canvas. Set it before judging anything:

   ```js
   import { PixiVfxRenderer, createPixiVfx2dProjection } from 'nixie-fx/pixi';

   const vfx = new PixiVfxRenderer({
     parent: app.stage,
     pixiRenderer: app.renderer,
     projection: createPixiVfx2dProjection({ originX: 240, originY: 250, pixelsPerUnit: 44, yAxis: 'up' }),
   });
   ```

2. **Expecting depth in Pixi.** World z only sets draw order there. Far flakes stay the same size, while Three.js shrinks them. If the 2D layer needs depth, use two emitters with different sizes and speeds.

3. **Using trails for rain.** Trails export as "partial". In Three.js they drew streaks but took 175 draw calls for 174 drops. In Pixi, at the same 0.03-unit width, the drops read as dots. Velocity-aligned billboards (`render.alignAxis: "velocity"`) with a tall start size (`startSizeSeparateAxes`, 0.025 × 0.22) are "supported" on both and stayed at 1 draw call.

4. **Leaving the trail width at its default.** The default is 1 world unit, so every drop turned into a big blob in both renderers.

5. **Checking ground collision too early.** At 4 s no flake had reached y = 0 yet, so the test said nothing. At 8 s, 32 flakes rested on the ground in both renderers and none were below it.

Not tested: textures, lighting, bloom, mobile, or anything beyond desktop Chrome with nixie-fx 0.1.20. Noise, collision and trails are all "partial", so rerun the same seed in both renderers after each upgrade.

If you tune weather in a particle editor for PixiJS and Three.js, the [NixieFX PixiJS particle effects guide](https://nixiefx.com/pixijs-particle-effects/) shows how to load one export in both.
