<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from "vue";

const SIZE = 15;
const SPEED = 200;
const INITIAL_SNAKE_SIZE = 3;
type GameState = "idle" | "playing" | "gameover";
type Direction = "up" | "down" | "left" | "right";

const generateInitialSnakeBody = () => {
  const result: string[] = [];
  const randomRow = Math.floor(Math.random() * SIZE) + 1;

  for (let cell = 0; cell < INITIAL_SNAKE_SIZE; cell++) {
    result.push(`${randomRow},${cell + 1}`);
  }

  return result;
};

const getRandomCell = () => {
  return `${Math.floor(Math.random() * SIZE) + 1},${Math.floor(
    Math.random() * SIZE + 1
  )}`;
};

const generateNextApple = () => {
  let nextApple = getRandomCell();
  while (snakeCells.value.includes(nextApple)) {
    nextApple = getRandomCell();
  }

  return nextApple;
};

const snakeCells = ref<string[]>(generateInitialSnakeBody());
const appleCell = ref<string>(generateNextApple());
let gameState: GameState = "idle";
let direction: Direction = "right";
let intervalId: number | null = null;

// prevent multiple keydown events in one interval
// that causes snake to go back on itself and die which is a bad UX
let inTransition = false;

const checkSnakeCell = (row: number, column: number) => {
  return snakeCells.value.includes(`${row},${column}`);
};

const checkAppleCell = (row: number, column: number) => {
  return appleCell.value === `${row},${column}`;
};

const getNewHead = () => {
  const head = snakeCells.value[snakeCells.value.length - 1];
  const [headRow, headColumn] = head.split(",").map(Number);
  let newHeadRow = headRow;
  let newHeadColumn = headColumn;

  if (direction === "right") {
    newHeadColumn = headColumn + 1;
  } else if (direction === "left") {
    newHeadColumn = headColumn - 1;
  } else if (direction === "up") {
    newHeadRow = headRow - 1;
  } else if (direction === "down") {
    newHeadRow = headRow + 1;
  }

  return `${newHeadRow},${newHeadColumn}`;
};

const checkValidNewHead = (newHead: string) => {
  const [newHeadRow, newHeadColumn] = newHead.split(",").map(Number);
  const isHitWall =
    newHeadRow < 1 ||
    newHeadRow > SIZE ||
    newHeadColumn < 1 ||
    newHeadColumn > SIZE;

  const isHitSnake = snakeCells.value.includes(newHead);
  if (isHitWall || isHitSnake) {
    return false;
  }

  return true;
};

const moveSnake = () => {
  const newHead = getNewHead();
  const isValidNewHead = checkValidNewHead(newHead);

  if (!isValidNewHead && intervalId) {
    clearInterval(intervalId);
    intervalId = null;
    gameState = "gameover";
    alert("Game Over");
    return;
  }

  // Reset transition flag
  inTransition = false;

  // add new head
  snakeCells.value.push(newHead);

  // check if apple eaten
  if (appleCell.value === newHead) {
    appleCell.value = generateNextApple();
    return;
  }

  // remove tail
  snakeCells.value.shift();
};

const startGame = () => {
  if (gameState == "playing") return;

  if (gameState === "gameover") {
    snakeCells.value = generateInitialSnakeBody();
    appleCell.value = generateNextApple();
    direction = "right";
    inTransition = false;
  }

  gameState = "playing";
  intervalId = setInterval(moveSnake, SPEED);
};

const currentScore = computed(() => {
  return snakeCells.value.length - INITIAL_SNAKE_SIZE;
});

const moveUp = () => {
  if (direction === "down") return;
  direction = "up";
};

const moveDown = () => {
  if (direction === "up") return;
  direction = "down";
};

const moveLeft = () => {
  if (direction === "right") return;
  direction = "left";
};

const moveRight = () => {
  if (direction === "left") return;
  direction = "right";
};

const handleKeyDown = (event: KeyboardEvent) => {
  if (gameState !== "playing") return;

  const handlers: Record<string, () => void> = {
    ArrowUp: moveUp,
    KeyW: moveUp,
    ArrowDown: moveDown,
    KeyS: moveDown,
    ArrowLeft: moveLeft,
    KeyA: moveLeft,
    ArrowRight: moveRight,
    KeyD: moveRight,
  };

  const handler = handlers[event.key];
  if (!inTransition && handler) {
    inTransition = true;
    handler();
  }
};

onMounted(() => {
  window.addEventListener("keydown", handleKeyDown);
});

onUnmounted(() => {
  if (intervalId) {
    clearInterval(intervalId);
  }

  window.removeEventListener("keydown", handleKeyDown);
});
</script>

<template>
  <button @click="startGame">Start Game</button>
  <p>Score: {{ currentScore }}</p>

  <div class="grid">
    <div v-for="row in SIZE" :key="row" class="row">
      <div
        v-for="column in SIZE"
        :key="column"
        class="cell"
        :class="{
          snake: checkSnakeCell(row, column),
          apple: checkAppleCell(row, column),
        }"
      ></div>
    </div>
  </div>
</template>

<style scoped>
.grid {
  margin-top: 16px;
  border: 0.5px solid black;
  width: fit-content;
}

.row {
  display: flex;
}

.cell {
  width: 30px;
  height: 30px;
  border: 0.5px solid black;
}

.snake {
  background-color: green;
}

.apple {
  background-color: red;
}
</style>
