<script setup lang="ts">
import { ref, computed } from 'vue'

type Player = 'X' | 'O' | null
const KING = 'X'
const QUEEN = 'O'

// State: 3x3 board, current turn, winner, status
const board = ref<Player[]>(Array(9).fill(null))
const turn = ref<Player>(KING)
const winner = ref<Player|null>(null)
const isDraw = ref(false)
const status = computed(() => {
  if (winner.value) {
    return `${winner.value === KING ? 'King (X)' : 'Queen (O)'} wins!`
  } else if (isDraw.value) {
    return "It's a draw!"
  } else {
    return `Turn: ${turn.value === KING ? 'King (X)' : 'Queen (O)'}`
  }
})

// For simple move animation, track animated cell
const lastMoveIndex = ref<number|null>(null)

// Winning combinations (row, col, diag)
const wins = [
  [0,1,2],[3,4,5],[6,7,8],
  [0,3,6],[1,4,7],[2,5,8],
  [0,4,8],[2,4,6],
]

// PUBLIC_INTERFACE
function handleCellClick(idx: number) {
  if (board.value[idx] || winner.value) return
  board.value[idx] = turn.value
  lastMoveIndex.value = idx

  // Animate only on move
  setTimeout(() => {
    lastMoveIndex.value = null
  }, 450)

  checkWinnerOrDraw()
  if (!winner.value && !isDraw.value) {
    turn.value = turn.value === KING ? QUEEN : KING
  }
}

// PUBLIC_INTERFACE
function restartGame() {
  board.value = Array(9).fill(null)
  turn.value = KING
  winner.value = null
  isDraw.value = false
  lastMoveIndex.value = null
}

// PUBLIC_INTERFACE
function renderPiece(piece: Player) {
  if (piece === KING) {
    // Unicode king, styled
    return `<span class="ttt-piece king" title="King (X)">&#9812;</span>`
  }
  if (piece === QUEEN) {
    // Unicode queen, styled
    return `<span class="ttt-piece queen" title="Queen (O)">&#9813;</span>`
  }
  return ''
}

// PUBLIC_INTERFACE
function checkWinnerOrDraw() {
  for (const combo of wins) {
    const [a,b,c] = combo
    if (board.value[a] && board.value[a] === board.value[b] && board.value[b] === board.value[c]) {
      winner.value = board.value[a]
      return
    }
  }
  if (board.value.every(cell => cell !== null) && !winner.value) {
    isDraw.value = true
  }
}
</script>

<template>
  <div class="ttt-root">
    <div class="ttt-statusbar">
      <span class="ttt-status"
        :class="{winner: !!winner, draw: isDraw}"
        >{{ status }}</span>
    </div>
    <div class="ttt-board" :class="{ended: winner||isDraw}">
      <div
        v-for="(cell, idx) in board"
        :key="idx"
        class="ttt-cell"
        :class="{
          filled: !!cell,
          [cell === 'X' ? 'is-king' : (cell === 'O' ? 'is-queen' : '')]: true,
          animated: lastMoveIndex === idx
        }"
        @click="handleCellClick(idx)"
        v-html="renderPiece(cell)"
        :aria-label="cell === 'X' ? 'King' : (cell === 'O' ? 'Queen' : 'Empty')"
      />
    </div>
    <div class="ttt-controls">
      <button class="ttt-btn" @click="restartGame">
        Restart Game
      </button>
    </div>
  </div>
</template>

<style scoped>
.ttt-root {
  padding: 2.5rem 2rem 2.5rem 2rem;
  background: #fff;
  border-radius: 1.25rem;
  box-shadow: 0 2px 16px 0 rgb(30 42 80 / 7%);
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 316px;
  min-height: 420px;
  max-width: 340px;
  margin: 0 auto;
}

/* Status bar */
.ttt-statusbar {
  width: 100%;
  text-align: center;
  margin-bottom: 1.7rem;
}
.ttt-status {
  font-size: 1.08rem;
  font-weight: 600;
  letter-spacing: 0.01em;
  color: var(--ttt-status-color, #444);
  transition: color .3s;
}
.ttt-status.winner {
  color: #4F46E5;
}
.ttt-status.draw {
  color: #F59E42;
}

/* Board styling */
.ttt-board {
  display: grid;
  grid-template-columns: repeat(3, 64px);
  grid-template-rows: repeat(3, 64px);
  gap: 0.7rem;
  margin-bottom: 2rem;
  pointer-events: auto;
  user-select: none;
  transition: opacity .24s;
}
.ttt-board.ended {
  opacity: .64;
  pointer-events: none;
}

/* Cells */
.ttt-cell {
  width: 64px;
  height: 64px;
  background: #f8fafd;
  border-radius: 1rem;
  border: 2.5px solid #f3f2ff;
  font-size: 2.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: box-shadow 0.24s, border-color .18s;
  box-shadow: 0 1px 6px #e4e9fa18;
  position: relative;
  outline: 0;
  will-change: filter;
}

.ttt-cell.filled {
  cursor: default;
}

.ttt-cell:hover:not(.filled):not(.animated) {
  border-color: #67C8FF;
  box-shadow: 0 0 0 3px #67C8FF1a;
}

.ttt-cell.animated {
  animation: popin 0.44s cubic-bezier(.4,2,.2,1.2);
  z-index: 1;
}

@keyframes popin {
  0%   { transform: scale(0.8); opacity: 0.7;}
  68%  { transform: scale(1.16); }
  90%  { transform: scale(0.98);}
  100% { transform: scale(1); opacity: 1;}
}

/* Piece style */
.ttt-piece {
  display: inline-block;
  font-family: 'Segoe UI Symbol','Arial',sans-serif;
  font-size: 2.1rem;
  font-weight: 800;
  line-height: 1;
  pointer-events: none;
}
.is-king .ttt-piece {
  background: linear-gradient(105deg, #4F46E5 60%, #67C8FF 100%);
  color: transparent;
  -webkit-background-clip: text;
  background-clip: text;
  filter: drop-shadow(0 2.5px 0.5px #d9e6ff88);
}
.is-queen .ttt-piece {
  background: linear-gradient(120deg, #F59E42 55%, #67C8FF 100%);
  color: transparent;
  -webkit-background-clip: text;
  background-clip: text;
  filter: drop-shadow(0 2.5px 0.5px #ffeed688);
}

/* Controls */
.ttt-controls {
  margin-top: 1.25em;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.ttt-btn {
  appearance: none;
  border: none;
  border-radius: .8em;
  font-size: 1rem;
  font-weight: 500;
  background: #4F46E5;
  color: #fff;
  padding: 0.69em 1.68em;
  box-shadow: 0 2px 8px 0 #4f47e524;
  cursor: pointer;
  outline: none;
  letter-spacing: 0.04em;
  margin-top: 0.1em;
  transition: background 0.18s, box-shadow .2s, color .22s;
}
.ttt-btn:hover, .ttt-btn:focus {
  background: #67C8FF;
  color: #fff;
}
</style>
