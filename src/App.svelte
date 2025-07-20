<script>
  import { onMount } from "svelte";
  import { Application } from "pixi.js";

  let canvas;

  onMount(async () => {
    window.__PIXI_DEVTOOLS__ = window.__PIXI_DEVTOOLS__ || {};

    const app = new Application();
    await app.init({ width: 300, height: 300, backgroundColor: 0x0000ff });
    canvas.appendChild(app.canvas);

    window.__PIXI_DEVTOOLS__.app = {
      stage: app.stage,
      renderer: app.renderer,
    };

    window.dispatchEvent(new Event("__PIXI_DEVTOOLS_APP_READY__"));
  });
</script>

<!-- Tailwind visual test -->
<div
  class="flex flex-col items-center justify-center min-h-screen bg-gray-900 text-white"
>
  <h1 class="text-3xl font-bold text-green-400 mb-4">✅ Tailwind + Svelte</h1>

  <div bind:this={canvas} class="border-4 border-yellow-400 rounded"></div>

  <p class="mt-4 text-sm text-blue-300">
    If you see a blue canvas above, Pixi.js is working ✅
  </p>
</div>
