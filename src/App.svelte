<script>
  import { onMount } from "svelte";
  import { Application, Assets, Sprite, Container, Graphics } from "pixi.js";

  let canvas;
  let app;
  const COLS = 5;
  const ROWS = 5;
  const TOTAL_COLS = 6; // includes god reel

  const cellSize = 124;
  const textures = {};
  const symbols = ["10", "jack", "queen", "king", "ace", "scatter"];
  const reelContainers = [];
  const expandedOverlays = []; // ✅ TRACK OVERLAYS TO REMOVE ON SPIN

  let grid;
  let godReel;

  async function loadSymbols() {
    const allSymbols = [
      "10",
      "jack",
      "queen",
      "king",
      "ace",
      "scatter",
      "reelexpanded",
      "odinreel",
      "heimdallreel",
      "freyareel",
      "thorreel",
      "idunreel",
      "xreel1",
      "xreel2",
      "xreel3",
    ];

    let loaded = 0;
    let failed = 0;

    for (const name of allSymbols) {
      try {
        textures[name] = await Assets.load(`/${name}.png`);
        console.log(`✅ Loaded: ${name}.png`);
        loaded++;
      } catch (e) {
        console.warn(`❌ Failed to load: ${name}.png`, e);
        failed++;
      }
    }

    console.log(`🧩 Symbols loaded: ${loaded}, Failed: ${failed}`);
  }

  function createReel(col) {
    const reel = new Container();
    reel.x = col * cellSize;
    reel.y = 0;
    reel.scrollY = 0;

    for (let i = 0; i < ROWS + 10; i++) {
      const key = symbols[Math.floor(Math.random() * symbols.length)];
      const sprite = new Sprite(textures[key]);
      sprite.anchor.set(0);
      sprite.width = sprite.height = cellSize;
      sprite.x = 0;
      sprite.y = i * cellSize;
      sprite.label = key;

      reel.addChild(sprite);
    }

    return reel;
  }

  function buildReels() {
    grid = new Container();

    for (let col = 0; col < COLS; col++) {
      const reel = createReel(col);
      grid.addChild(reel);
      reelContainers[col] = reel;
    }

    // ✅ Add this here — God Reel (column 5)
    godReel = new Container();
    godReel.x = COLS * cellSize + 65; // adjust '20' as needed

    godReel.y = 0;

    const godSymbols = [
      "xreel1",
      "xreel2",
      "xreel3",
      "odinreel",
      "heimdallreel",
      "freyareel",
      "thorreel",
      "idunreel",
    ];

    for (let i = 0; i < 20; i++) {
      const key = godSymbols[Math.floor(Math.random() * godSymbols.length)];
      const sprite = new Sprite(textures[key]);
      sprite.anchor.set(0.5);
      sprite.width = 124;
      sprite.height = 620;
      sprite.x = 0;
      sprite.y = i * 620;
      sprite.name = key;
      godReel.addChild(sprite);
    }

    godReel.visible = true;
    grid.addChild(godReel);

    grid.x = 440;
    grid.y = 195;

    const mask = new Graphics();
    mask.fill({ color: 0xffffff });
    mask.rect(0, 0, TOTAL_COLS * cellSize, ROWS * cellSize);

    mask.fill();
    mask.x = grid.x - 0;
    mask.y = grid.y - 0;
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
            sprite.name = newKey;
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

  let reelsFinished = 0;
  function finalizeReel(col) {
    const reel = reelContainers[col];
    reel.removeChildren();

    for (let row = 0; row < ROWS + 10; row++) {
      const key = symbols[Math.floor(Math.random() * symbols.length)];
      const sprite = new Sprite(textures[key]);
      sprite.anchor.set(0);
      sprite.width = sprite.height = cellSize;
      sprite.x = 0;
      sprite.y = row * cellSize;
      sprite.name = key;
      reel.addChild(sprite);
    }

    reel.y = 0;

    reelsFinished++;
    if (reelsFinished === COLS) {
      reelsFinished = 0;
      setTimeout(triggerScatterAnimations, 300);
    }
  }

  function triggerScatterAnimations() {
    for (let col = 0; col < COLS; col++) {
      const reel = reelContainers[col];

      for (const sprite of reel.children) {
        if (
          sprite.name === "scatter" &&
          sprite.y >= 0 &&
          sprite.y + cellSize <= cellSize * ROWS
        ) {
          pulseAndExpand(sprite, col); // ✅ only first visible scatter per reel
          break;
        }
      }
    }
  }

  function pulseAndExpand(sprite, col) {
    let pulseCount = 0;
    let scaleDir = 1;

    const pulse = () => {
      sprite.scale.x += 0.03 * scaleDir;
      sprite.scale.y += 0.03 * scaleDir;

      if (sprite.scale.x >= 1) scaleDir = -1;
      if (sprite.scale.x <= 1 && scaleDir === -1) {
        pulseCount++;
        if (pulseCount < 2) {
          scaleDir = 1;
        } else {
          sprite.scale.set(1);
          expandReel(col);
          return;
        }
      }

      requestAnimationFrame(pulse);
    };

    pulse();
  }

  function expandReel(col) {
    const expanded = new Sprite(textures["reelexpanded"]);
    expanded.anchor.set(0.5); // ✅ center aligned
    expanded.x = col * cellSize + cellSize / 2;
    expanded.y = (cellSize * ROWS) / 2;
    expanded.width = cellSize;
    expanded.height = cellSize * ROWS;
    expanded.alpha = 0;
    expanded.zIndex = 100;

    grid.addChild(expanded);
    expandedOverlays.push(expanded); // ✅ track it

    app.ticker.add(() => {
      if (expanded.alpha < 1) expanded.alpha += 0.05;
    });
  }
  function handleSpin() {
    // ✅ Remove previous overlays
    for (const overlay of expandedOverlays) {
      overlay.destroy(); // or: grid.removeChild(overlay)
    }
    expandedOverlays.length = 0;

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

<!-- PIXI Canvas -->
<div bind:this={canvas} class="absolute top-0 left-0 w-full h-full z-0"></div>

<!-- UI Layer -->
<div class="absolute z-10 pointer-events-none w-full h-full">
  <div
    class="absolute pointer-events-auto text-white text-xl font-bold bg-black/70 px-4 py-2 rounded"
    style="left: 1290px; top: 778px; width: 614px; height: 122px;"
  >
    BALANCE: $1,000.00
  </div>

  <div
    class="absolute pointer-events-auto text-white text-lg bg-blue-800 px-12 py-2 rounded"
    style="left: 20px; top: 20px; width: 175px; height: 48px;"
  >
    ☰ MENU
  </div>

  <button
    on:click={handleSpin}
    class="absolute pointer-events-auto bg-green-500 hover:bg-blue-800 text-white font-bold rounded-full shadow-lg text-2xl"
    style="left: 1446px; top: 452px; width: 100px; height: 100px;"
  >
    SPIN
  </button>

  <div
    class="absolute border-4 border-white rounded-lg pointer-events-none"
    style="left: 435px; top: 198px; width: 753px; height: 620px;"
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
