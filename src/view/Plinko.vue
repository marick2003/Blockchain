<template>
    <div class="plinko-container">
      <canvas ref="canvas" class="game-canvas"></canvas>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, onMounted, onUnmounted } from "vue";
  import Matter from "matter-js";
  
  const canvas = ref<HTMLCanvasElement | null>(null);
  
  onMounted(() => {
    const { Engine, Render, Runner, Bodies, Composite, Events } = Matter;
  
    // 基本設定
    const WIDTH = 800;
    const HEIGHT = 600;
    const engine = Engine.create();
    const world = engine.world;
  
    // 設定渲染器
    const render = Render.create({
      canvas: canvas.value as HTMLCanvasElement,
      engine: engine,
      options: {
        width: WIDTH,
        height: HEIGHT,
        background: "#0f1728",
        wireframes: false,
      },
    });
    Render.run(render);
  
    // 啟動運行器
    const runner = Runner.create();
    Runner.run(runner, engine);
  
    // **放置牆壁**
    const walls = [
      Bodies.rectangle(WIDTH / 2, 0, WIDTH, 20, { isStatic: true }),
      Bodies.rectangle(WIDTH / 2, HEIGHT, WIDTH, 20, { isStatic: true }),
      Bodies.rectangle(0, HEIGHT / 2, 20, HEIGHT, { isStatic: true }),
      Bodies.rectangle(WIDTH, HEIGHT / 2, 20, HEIGHT, { isStatic: true }),
    ];
    Composite.add(world, walls);
  
    // **放置釘子**
    const pinRadius = 5;
    const rows = 10;
    const cols = 8;
    const pinSpacing = 70;
  
    for (let row = 0; row < rows; row++) {
      for (let col = 0; col < cols; col++) {
        const xOffset = row % 2 === 0 ? pinSpacing / 2 : 0; // 錯位排列
        const x = col * pinSpacing + xOffset + 100;
        const y = row * pinSpacing + 50;
  
        const pin = Bodies.circle(x, y, pinRadius, {
          isStatic: true,
          render: {
            fillStyle: "#ffffff",
          },
        });
        Composite.add(world, pin);
      }
    }
  
    // **放置分區**
    const binCount = cols + 1;
    const binWidth = WIDTH / binCount;
  
    for (let i = 0; i < binCount; i++) {
      const bin = Bodies.rectangle(
        i * binWidth,
        HEIGHT - 50,
        10,
        100,
        {
          isStatic: true,
          render: {
            fillStyle: "#ccc",
          },
        }
      );
      Composite.add(world, bin);
    }
  
    // **點擊添加球體**
    const ballRadius = 10;
    const addBall = (x: number) => {
      const ball = Bodies.circle(x, 10, ballRadius, {
        restitution: 0.8,
        friction: 0.01,
        render: {
          fillStyle: "#ff5722",
        },
      });
      Composite.add(world, ball);
    };
  
    document.addEventListener("click", (event) => {
      if (canvas.value) {
        const rect = canvas.value.getBoundingClientRect();
        const x = event.clientX - rect.left;
        addBall(x);
      }
    });
  
    // **移除掉出畫布的球體**
    Events.on(engine, "afterUpdate", () => {
      const bodies = Composite.allBodies(world);
      bodies.forEach((body) => {
        if (body.position.y > HEIGHT + 100) {
          Composite.remove(world, body);
        }
      });
    });
  
    onUnmounted(() => {
      Render.stop(render);
      Runner.stop(runner);
      Composite.clear(world);
      Engine.clear(engine);
    });
  });
  </script>
  
  <style>
  .plinko-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background: #222;
  }
  .game-canvas {
    border: 1px solid #ccc;
  }
  </style>
  