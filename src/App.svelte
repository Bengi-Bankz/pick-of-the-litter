<script>
  import { onMount } from "svelte";
  import { Application, Assets, Sprite, Container, Graphics } from "pixi.js";

  let canvas;
  let app;
  const COLS = 6;
  const ROWS = 5;
  const cellSize = 124;
  const textures = {};
  const symbols = ["10", "jack", "queen", "king", "ace"];
  const reelContainers = [];

  async function loadSymbols() {
    for (const name of symbols) {
      textures[name] = await Assets.load(`/${name}.png`);
    }
  }

  function createReel(col) {
    const reel = new Container();
    reel.x = col * cellSize;
    reel.y = 0;
    reel.scrollY = 0;

    for (let i = 0; i < ROWS + 10; i++) {
      const key = symbols[Math.floor(Math.random() * symbols.length)];
      const sprite = new Sprite(textures[key]);
      sprite.anchor.set(0.5);
      sprite.width = sprite.height = cellSize;
      sprite.x = 0;
      sprite.y = i * cellSize;
      reel.addChild(sprite);
    }

    return reel;
  }

  function buildReels() {
    const grid = new Container();

    for (let col = 0; col < COLS; col++) {
      const reel = createReel(col);
      grid.addChild(reel);
      reelContainers[col] = reel;
    }

    // Align perfectly with overlay (optional)
    grid.x = 642;
    grid.y = 105;

    // Create a mask to show only 5x6 visible area
    const mask = new Graphics();
    mask.beginFill(0xffffff);
    mask.drawRect(0, 0, COLS * cellSize, ROWS * cellSize);
    mask.endFill();
    mask.x = grid.x - 60;
    mask.y = grid.y - 60;
    app.stage.addChild(mask);

    grid.mask = mask;
    app.stage.addChild(grid);
  }

  function spinReels() {
    for (let col = 0; col < COLS; col++) {
      const reel = reelContainers[col];
      const spinTime = 1000 + col * 300;
      const stopTime = Date.now() + spinTime;
      const totalHeight = cellSize * (ROWS + 10);
      const scrollSpeed = 60;

      function animate() {
        reel.scrollY += scrollSpeed;

        reel.children.forEach((sprite) => {
          sprite.y += scrollSpeed;
          if (sprite.y > totalHeight) {
            sprite.y -= totalHeight;
            const newKey = symbols[Math.floor(Math.random() * symbols.length)];
            sprite.texture = textures[newKey];
          }
        });

        if (Date.now() < stopTime) {
          requestAnimationFrame(animate);
        } else {
          finalizeReel(col);
        }
      }

      animate();
    }
  }

  function finalizeReel(col) {
    const reel = reelContainers[col];
    reel.removeChildren();

    for (let row = 0; row < ROWS + 10; row++) {
      const key = symbols[Math.floor(Math.random() * symbols.length)];
      const sprite = new Sprite(textures[key]);
      sprite.anchor.set(0.5);
      sprite.width = sprite.height = cellSize;
      sprite.x = 0;
      sprite.y = row * cellSize;
      reel.addChild(sprite);
    }

    reel.y = 0;
  }

  function handleSpin() {
    spinReels();
  }

  onMount(async () => {
    app = new Application();
    await app.init({
      width: window.innerWidth,
      height: window.innerHeight,
      backgroundColor: 0x000000,
      resolution: window.devicePixelRatio || 1,
      autoDensity: true,
    });

    canvas.appendChild(app.canvas);
    window.__PIXI_DEVTOOLS__ = { app };

    window.addEventListener("resize", () => {
      app.renderer.resize(window.innerWidth, window.innerHeight);
    });

    await loadSymbols();
    buildReels();
  });
</script>

<!-- 🎯 PIXI Canvas -->
<div bind:this={canvas} class="absolute top-0 left-0 w-full h-full z-0"></div>

<!-- 🎛️ UI Layer -->
<div class="absolute z-10 pointer-events-none w-full h-full">
  <!-- 💰 Balance -->
  <div
    class="absolute pointer-events-auto text-white text-xl font-bold bg-black/70 px-4 py-2 rounded"
    style="left: 1290px; top: 778px; width: 614px; height: 122px;"
  >
    BALANCE: $1,000.00
  </div>

  <!-- 📋 Menu -->
  <div
    class="absolute pointer-events-auto text-white text-sm bg-gray-800 px-3 py-2 rounded"
    style="left: 0px; top: 0px; width: 251px; height: 88px;"
  >
    ☰ MENU
  </div>

  <!-- 🌀 Spin Button -->
  <button
    on:click={handleSpin}
    class="absolute pointer-events-auto bg-green-500 hover:bg-green-600 text-white font-bold rounded-full shadow-lg text-2xl"
    style="left: 1446px; top: 452px; width: 100px; height: 100px;"
  >
    SPIN
  </button>

  <!-- 🖼️ Frame Overlay -->
  <div
    class="absolute border-4 border-white rounded-lg pointer-events-none"
    style="left: 575px; top: 45px; width: 753px; height: 620px;"
  ></div>
</div>

<style>
  :global(html),
  :global(body) {
    margin: 0;
    padding: 0;
    overflow: hidden;
    background-color: black;
  }
</style>
