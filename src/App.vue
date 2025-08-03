<script setup lang="ts">
import { onMounted, reactive, ref } from "vue";
import asafmazon from "./assets/asafmazon.png";
import rifle from "./assets/assault-rifle-icon-flat-vector-23457938-removebg-preview.png";

const score = ref(0);
const gameArea = ref<HTMLDivElement | null>(null);
const rifleRef = ref<HTMLImageElement | null>(null);
const mouse = reactive({ x: 0, y: 0 });
const angle = ref(0);
const bullets = ref<{ x: number; y: number; dx: number; dy: number }[]>([]);

const cats = reactive(
  Array.from({ length: 6 }).map(() => ({
    x: Math.random() * 500,
    y: Math.random() * 300,
    dx: (Math.random() - 0.5) * 10,
    dy: (Math.random() - 0.5) * 10,
    size: 40 + Math.random() * 40,
    src: `https://cataas.com/cat?${Math.random()}`,
  }))
);

function updateMouse(e: MouseEvent) {
  if (!gameArea.value || !rifleRef.value) return;

  const gunBounds = rifleRef.value.getBoundingClientRect();
  mouse.x = e.clientX;
  mouse.y = e.clientY;

  const gunX = gunBounds.left + gunBounds.width / 2;
  const gunY = gunBounds.top + gunBounds.height / 2;

  angle.value = Math.atan2(mouse.y - gunY, mouse.x - gunX);
}

let lastShotTime = 0;

function shoot() {
  console.log("shoot clicked");
  const now = Date.now();
  if (now - lastShotTime < 200) return;
  lastShotTime = now;

  if (!rifleRef.value || !gameArea.value) return;
  const gameBounds = gameArea.value.getBoundingClientRect();
  const gunBounds = rifleRef.value.getBoundingClientRect();
  const barrelLength = gunBounds.width / 2;

  const gunX =
    gunBounds.left + gunBounds.width / 2 + Math.cos(angle.value) * barrelLength;
  const gunY =
    gunBounds.top + gunBounds.height / 2 + Math.sin(angle.value) * barrelLength;

  bullets.value.push({
    x: gunX - gameBounds.left,
    y: gunY - gameBounds.top,
    dx: Math.cos(angle.value) * 5,
    dy: Math.sin(angle.value) * 5,
  });
  console.log(bullets.value);
}

function animate() {
  const bounds = gameArea.value?.getBoundingClientRect();
  if (!bounds) return;

  for (const cat of cats) {
    cat.x += cat.dx;
    cat.y += cat.dy;

    if (cat.x <= 0) {
      cat.x = 0;
      cat.dx *= -1;
    }

    if (cat.x + cat.size >= bounds.width) {
      cat.x = bounds.width - cat.size;
      cat.dx *= -1;
    }

    if (cat.y <= 0) {
      cat.y = 0;
      cat.dy *= -1;
    }

    if (cat.y + cat.size >= bounds.height) {
      cat.y = bounds.height - cat.size;
      cat.dy *= -1;
    }
  }

  for (let i = bullets.value.length - 1; i >= 0; i--) {
    const b = bullets.value[i];
    b.x += b.dx;
    b.y += b.dy;

    // Out of bounds check
    if (b.y < 0 || b.x > bounds.width || b.y > bounds.height) {
      bullets.value.splice(i, 1);
      continue;
    }

    // Collision with cats
    for (let j = 0; j < cats.length; j++) {
      const c = cats[j];
      if (
        b.x >= c.x &&
        b.x <= c.x + c.size &&
        b.y >= c.y &&
        b.y <= c.y + c.size
      ) {
        // Hit!
        bullets.value.splice(i, 1);
        score.value++;
        cats[j] = {
          ...c,
          x: Math.random() * (bounds.width - 80),
          y: Math.random() * (bounds.height - 80),
          src: `https://cataas.com/cat?${Math.random()}`,
        };
        break;
      }
    }
  }

  requestAnimationFrame(animate);
}

onMounted(() => {
  animate();
  const bounds = gameArea.value?.getBoundingClientRect();
  console.log(bounds);
});
</script>

<template>
  <div class="h-full flex">
    <!-- left container -->
    <div class="h-full w-[25%] border-r-3">
      <h1 class="text-center text-3xl mt-20">Score: {{ score }}</h1>
      <div class="h-full">
        <div>
          <img :src="asafmazon" alt="asafmazon" class="mt-100 relative" />
        </div>

        <img
          ref="rifleRef"
          :src="rifle"
          alt="rifle"
          :style="{ transform: `rotate(${angle}rad)` }"
          class="absolute top-150 size-80"
        />
        />
      </div>
    </div>

    <!-- Game Area -->
    <div
      ref="gameArea"
      class="relative w-[75%] h-screen bg-red-300"
      @mousemove="updateMouse"
      @click="shoot"
    >
      <div
        v-for="(bullet, i) in bullets"
        :key="'b' + i"
        :style="{ left: bullet.x + 'px', top: bullet.y + 'px' }"
        class="absolute w-2 h-2 bg-yellow-400 rounded-full"
      ></div>
      <img
        v-for="(cat, index) in cats"
        :key="index"
        :src="cat.src"
        :style="{
          position: 'absolute',
          top: cat.y + 'px',
          left: cat.x + 'px',
          width: cat.size + 'px',
          height: 'auto',
        }"
        class="pointer-events-none"
      />
    </div>
  </div>
</template>
