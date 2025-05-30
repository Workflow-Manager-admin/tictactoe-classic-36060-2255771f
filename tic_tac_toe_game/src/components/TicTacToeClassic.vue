<script setup lang="ts">
import { ref, computed } from "vue";

// Board is a 3x3 array, null = empty, "X" or "O"
const board = ref([
  [null, null, null],
  [null, null, null],
  [null, null, null],
]);

const xGoesFirst = ref(true);
const currentPlayer = computed(() => (xGoesFirst.value
  ? (moveCount.value % 2 === 0 ? "X" : "O")
  : (moveCount.value % 2 === 0 ? "O" : "X"))
);
const gameOver = ref(false);
const winner = ref(null);
const moveCount = computed(() =>
  board.value.flat().filter((x) => x !== null).length
);

// PUBLIC_INTERFACE
function handleCellClick(row, col) {
  if (gameOver.value || board.value[row][col]) return;
  board.value[row][col] = currentPlayer.value;
  checkGameState();
}

// PUBLIC_INTERFACE
function resetGame() {
  board.value = [
    [null, null, null],
    [null, null, null],
    [null, null, null],
  ];
  winner.value = null;
  gameOver.value = false;
}

// PUBLIC_INTERFACE
function switchFirstPlayer() {
  xGoesFirst.value = !xGoesFirst.value;
  resetGame();
}

// PUBLIC_INTERFACE
function checkGameState() {
  // Rows, columns, diagonals
  const lines = [
    // Rows
    ...board.value,
    // Columns
    [0, 1, 2].map((i) => [board.value[0][i], board.value[1][i], board.value[2][i]]),
    // Diagonal TL-BR
    [board.value[0][0], board.value[1][1], board.value[2][2]],
    // Diagonal BL-TR
    [board.value[2][0], board.value[1][1], board.value[0][2]],
  ];
  for (const line of lines) {
    if (
      line[0] &&
      line[0] === line[1] &&
      line[1] === line[2]
    ) {
      winner.value = line[0];
      gameOver.value = true;
      return;
    }
  }
  const draw = board.value.flat().every((cell) => cell !== null);
  if (draw) {
    gameOver.value = true;
    winner.value = null;
  }
}

const statusMessage = computed(() => {
  if (gameOver.value) {
    if (winner.value) return `Winner: ${winner.value} 🎉`;
    return "It's a draw! 🤝";
  }
  return `Current Turn: ${currentPlayer.value}`;
});
</script>

<template>
  <div class="ttt-container">
    <h1 class="title">TicTacToe Classic</h1>
    <div class="ttt-board">
      <div
        v-for="(row, rIdx) in board"
        :key="'row'+rIdx"
        class="ttt-row"
      >
        <button
          v-for="(cell, cIdx) in row"
          :key="'cell'+cIdx"
          class="ttt-cell"
          :disabled="!!cell || gameOver"
          @click="handleCellClick(rIdx, cIdx)"
          :style="{
            color:
              cell === 'X'
                ? 'var(--ttt-x-color)'
                : cell === 'O'
                ? 'var(--ttt-o-color)'
                : 'var(--color-text)',
          }"
        >
          {{ cell || "" }}
        </button>
      </div>
    </div>
    <div class="ttt-status" :class="{ over: gameOver }">
      {{ statusMessage }}
    </div>
    <div class="ttt-controls">
      <button class="ttt-reset" @click="resetGame">
        Reset Game
      </button>
      <button
        class="ttt-switch"
        @click="switchFirstPlayer"
        title="Start New Game With Other Player First"
      >
        Switch First Player (Now: {{ xGoesFirst ? "X" : "O" }})
      </button>
    </div>
  </div>
</template>

<style scoped>
/* Theme colors */
:root {
  --ttt-primary: #ffffff;
  --ttt-secondary: #000000;
  --ttt-accent: #2196f3;
  --ttt-x-color: var(--ttt-accent);
  --ttt-o-color: var(--ttt-secondary);
}
.ttt-container {
  background: var(--ttt-primary);
  color: var(--ttt-secondary);
  margin: 0 auto;
  max-width: 340px;
  min-height: 470px;
  padding: 2rem 1.5rem 1.5rem 1.5rem;
  border-radius: 16px;
  box-shadow: 0 2px 16px rgba(33, 150, 243, 0.07),0 1.5px 9px rgba(0,0,0,0.07);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.title {
  text-align: center;
  font-size: 2rem;
  font-weight: 700;
  letter-spacing: 1px;
  color: var(--ttt-accent);
  margin-bottom: 1rem;
}
.ttt-board {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 1.25rem;
}
.ttt-row {
  display: flex;
}
.ttt-cell {
  background: var(--ttt-primary);
  border: 2px solid var(--ttt-accent);
  width: 64px;
  height: 64px;
  margin: 4px;
  font-size: 2.3rem;
  font-family: inherit;
  font-weight: 600;
  outline: none;
  cursor: pointer;
  border-radius: 8px;
  transition: background 0.2s, color 0.1s, border 0.1s;
  user-select: none;
  box-shadow: 0 0.5px 3px rgba(33, 150, 243, 0.05);
}
.ttt-cell:disabled {
  opacity: 0.65;
  cursor: default;
  background: #efefef;
}
.ttt-cell:hover:not(:disabled) {
  background: #e3f2fd;
}
.ttt-status {
  font-size: 1.15rem;
  padding: 0.6rem 0;
  text-align: center;
  min-height: 1.8em;
  color: var(--ttt-secondary);
  letter-spacing: 0.5px;
}
.ttt-status.over {
  color: var(--ttt-accent);
  font-weight: bold;
}
.ttt-controls {
  display: flex;
  gap: 1rem;
  margin-top: 0.6rem;
}
.ttt-reset, .ttt-switch {
  padding: 7px 18px;
  font-size: 1rem;
  border-radius: 6px;
  border: none;
  outline: none;
  font-family: inherit;
  cursor: pointer;
  color: var(--ttt-primary);
  background: var(--ttt-accent);
  transition: background 0.2s, color 0.12s;
}
.ttt-reset:hover, .ttt-switch:hover {
  background: #1769aa;
}
.ttt-switch {
  background: var(--ttt-secondary);
  color: var(--ttt-primary);
}

@media (max-width: 450px) {
  .ttt-container {
    min-width: 92vw;
    max-width: 95vw;
    padding: 1.2rem 0.5rem;
  }
  .ttt-cell {
    width: 17vw;
    height: 17vw;
    min-width: 42px;
    min-height: 42px;
    font-size: 1.2rem;
  }
}
</style>
