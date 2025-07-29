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
  background: var(--color-background);
  border-radius: 1.25rem;
  box-shadow: 0 2px 16px 0 rgb(30 42 80 / 10%);
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 316px;
  min-height: 420px;
  max-width: 340px;
  margin: 0 auto;
  box-shadow: 0 2px 14px 0 #2e1a4717;
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
  color: var(--color-primary);
  transition: color .3s;
}
.ttt-status.winner {
  color: var(--color-accent);
  text-shadow: 0 1px 0 #2222, 0 2px 4px #fff0;
}
.ttt-status.draw {
  color: var(--color-secondary);
  text-shadow: 0 1px 0 #2221, 0 2px 3px #fff2;
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
  border-radius: 1.25rem;
}
.ttt-board.ended {
  opacity: .64;
  pointer-events: none;
}

/* Cells */
.ttt-cell {
  width: 64px;
  height: 64px;
  background: var(--color-background-soft); /* Near white ADA bg */
  border-radius: 1rem;
  border: 2.5px solid var(--color-border);
  font-size: 2.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: box-shadow 0.19s, border-color .16s, background .28s;
  box-shadow: 0 1px 5px #2e1a470b;
  position: relative;
  outline: 0;
  will-change: filter;
  color: var(--color-text-strong);
}

.ttt-cell.filled {
  cursor: default;
}

.ttt-cell:hover:not(.filled):not(.animated),
.ttt-cell:focus-visible:not(.filled) {
  border-color: var(--color-accent);
  box-shadow: 0 0 0 3px #ffd6002b;
  background: #fffde7;
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

/* Piece style (accessibility: high contrast) */
.ttt-piece {
  display: inline-block;
  font-family: 'Segoe UI Symbol','Arial',sans-serif;
  font-size: 2.08rem;
  font-weight: 900;
  line-height: 1;
  pointer-events: none;
}

.is-king .ttt-piece {
  /* Deep dark purple text, yellow highlight, ADA-contrasted */
  background: linear-gradient(110deg, #2e1a47 82%, #ffd600 100%);
  color: transparent;
  -webkit-background-clip: text;
  background-clip: text;
  filter: drop-shadow(0 1.5px 0.5px #ffd60033) drop-shadow(0 0.5px 0.1px #1111);
  text-shadow: 0 0.5px 1.5px #fff8, 0 2px 4px #2222;
}

.is-queen .ttt-piece {
  /* Deep gold, ADA contrasted */
  background: linear-gradient(100deg, #ffd600 64%, #2e1a47 100%);
  color: transparent;
  -webkit-background-clip: text;
  background-clip: text;
  filter: drop-shadow(0 2px 1.5px #2e1a4744) drop-shadow(0 0.5px 0.1px #ffd60099);
  text-shadow: 0 0.5px 1px #2224, 0 2.5px 8px #ffd60033;
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
  font-weight: 600;
  background: var(--color-primary);
  color: #fff;
  padding: 0.69em 1.7em;
  box-shadow: 0 1.5px 6px 0 #2222;
  cursor: pointer;
  outline: none;
  letter-spacing: 0.06em;
  margin-top: 0.18em;
  border: 2px solid var(--color-accent);
  transition: 
    background-color .17s,
    border .15s,
    color .19s,
    box-shadow .18s;
}
.ttt-btn:hover, .ttt-btn:focus {
  background: var(--color-accent);
  color: var(--color-primary);
  border: 2.5px solid var(--color-primary);
  box-shadow: 0 3px 12px 0 #ffd60030;
}

/* Focus ring for accessibility */
.ttt-cell:focus-visible, .ttt-btn:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 1px;
}
</style>
