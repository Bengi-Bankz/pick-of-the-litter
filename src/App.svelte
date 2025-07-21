<script>
  import { onMount } from "svelte";
  import { Application, Assets, AnimatedSprite } from "pixi.js";

  let canvas;

  onMount(async () => {
    const app = new Application();
    await app.init({
      width: window.innerWidth,
      height: window.innerHeight,
      backgroundColor: 0x000000,
      resolution: window.devicePixelRatio || 1,
      autoDensity: true,
    });

    canvas.appendChild(app.canvas);

    // 🔄 Resize on window resize
    window.addEventListener("resize", () => {
      app.renderer.resize(window.innerWidth, window.innerHeight);
    });

    // 📦 Load gridframe atlas
    await Assets.load("/gridframe.png.json");

    // 🎞 Collect frames from frame_001 to frame_005
    const textures = [];
    for (let i = 1; i <= 5; i++) {
      const index = i.toString().padStart(3, "0");
      textures.push(Assets.get(`frame_${index}.png`));
    }

    // 🌀 Create and configure animation
    const anim = new AnimatedSprite(textures);
    anim.anchor.set(0.5);
    anim.x = app.screen.width / 2;
    anim.y = app.screen.height / 2;
    anim.animationSpeed = 0.15;
    anim.loop = true;
    anim.play();

    // 🔍 Optional: Scale down if too big for viewport
    if (anim.width > app.screen.width || anim.height > app.screen.height) {
      const scaleX = app.screen.width / anim.width;
      const scaleY = app.screen.height / anim.height;
      anim.scale.set(Math.min(scaleX, scaleY) * 0.9);
    }

    app.stage.addChild(anim);
  });
</script>

<div bind:this={canvas}></div>

<style>
  html,
  body {
    margin: 0;
    padding: 0;
    overflow: hidden;
    background-color: black;
  }
</style>
