<template>
    <div class="h-screen flex items-center justify-center bg-[#14120B] text-[#F2F2F2]">
      <div class="flex flex-col items-center gap-12 py-8 px-4 w-full max-w-md md:max-w-lg">
        <h1 class="text-2xl text-center font-black bg-gradient-to-r from-[#F2F2F2] via-[#A8A8A8] to-[#F2F2F2] bg-clip-text text-transparent drop-shadow-2xl">
          Крестики-Нолики
        </h1>
        
        <div class="text-lg font-semibold text-center px-6 py-2 bg-[#1A170F]/50 backdrop-blur-sm rounded-xl border border-[#333]/50 shadow-xl min-w-[250px]">
          {{ status }}
        </div>
        
        <div class="relative bg-[#1A170F] rounded-2xl p-6 shadow-2xl shadow-black/50 border border-[#333]/30 w-full aspect-square grid grid-cols-3 grid-rows-3 gap-4 overflow-hidden">
          <div
            v-for="n in 9"
            :key="n"
            :class="[
              'aspect-square bg-[#0F0D08]/80 rounded-xl flex items-center justify-center text-4xl md:text-5xl font-black cursor-pointer transition-all duration-300 ease-out hover:scale-105 hover:bg-[#201B10]/80 active:scale-95 shadow-md border-2 border-transparent group',
              {
                'text-[#00FF88] shadow-[0_0_20px_#00FF88_/_0.4] border-[#00FF88]/30 bg-[#00FF88]/5 hover:shadow-[0_0_30px_#00FF88_/_0.6]': board[n-1] === 'X',
                'text-[#FF0088] shadow-[0_0_20px_#FF0088_/_0.4] border-[#FF0088]/30 bg-[#FF0088]/5 hover:shadow-[0_0_30px_#FF0088_/_0.6]': board[n-1] === 'O',
                '!cursor-default pointer-events-none scale-100 group-hover:scale-100': winner !== null
              }
            ]"
            @click="makeMove(n-1)"
          >
            <span class="transition-all duration-300 scale-0 group-hover:scale-110 origin-center" :class="{ 'scale-100': board[n-1] !== null }">
              {{ board[n-1] }}
            </span>
          </div>
  
          <!-- Неоновая линия победы в цвете победителя -->
          <transition name="line">
            <div v-if="winner && winner !== 'draw'" class="absolute inset-0 pointer-events-none">
              <div :class="winningLineClass"></div>
            </div>
          </transition>
        </div>
        
        <div class="flex items-center gap-2">
          <button
            @click="reset"
            class="px-6 py-2 bg-[#F2F2F2]/10 text-[#F2F2F2] font-bold text-lg rounded-2xl hover:bg-[#F2F2F2]/20 active:bg-[#F2F2F2]/5 transition-all duration-300 shadow-lg shadow-black/50 hover:shadow-xl hover:shadow-black/70 border border-[#333]/50 focus:outline-none focus:ring-2 focus:ring-[#A8A8A8]/50"
          >
            {{ winner === null ? 'Новая игра' : 'Играть снова' }}
          </button>
          <button @click="passwordStore.password = ''"
            class="px-6 py-2 bg-[#F2F2F2]/10 text-[#F2F2F2] font-bold text-lg rounded-2xl hover:bg-[#F2F2F2]/20 active:bg-[#F2F2F2]/5 transition-all duration-300 shadow-lg shadow-black/50 hover:shadow-xl hover:shadow-black/70 border border-[#333]/50 focus:outline-none focus:ring-2 focus:ring-[#A8A8A8]/50"
          >
            В меню
          </button>
        </div>
      </div>
    </div>
  </template>
  
  <script setup lang="ts">  
  const board = ref<(string | null)[]>(Array(9).fill(null))
  const currentPlayer = ref<'X' | 'O'>('X')
  const winner = ref<string | null>(null)
  const status = ref<string>('Ваш ход (X)')
  const lossStreak = ref(0)
  const mistakeChance = computed(() => Math.min(lossStreak.value * 0.25, 0.75))
  
  const winningCombo = ref<number[]>([])
  
  const getWinner = (b: (string | null)[]): { winner: string | null, combo?: number[] } => {
    const lines = [
      [0, 1, 2], [3, 4, 5], [6, 7, 8],
      [0, 3, 6], [1, 4, 7], [2, 5, 8],
      [0, 4, 8], [2, 4, 6]
    ]
    for (const line of lines) {
      const [a, bIdx, c] = line
      if (b[a] && b[a] === b[bIdx] && b[a] === b[c]) {
        return { winner: b[a]!, combo: line }
      }
    }
    const isFull = b.every(cell => cell !== null)
    return { winner: isFull ? 'draw' : null }
  }
  
  const winningLineClass = computed(() => {
    if (!winningCombo.value.length) return ''
    const [a, , c] = winningCombo.value
    const color = winner.value === 'X' ? '#00FF88' : '#FF0088'
    const base = `absolute bg-${color} shadow-[0_0_40px_${color}_/_0.9] rounded-full opacity-80`
  
    if (a === 0 && c === 2) return `${base} top-[calc(16.666%+8px)] left-0 w-full h-5`           // row 1
    if (a === 3 && c === 5) return `${base} top-1/2 left-0 w-full h-5 -translate-y-1/2`         // row 2
    if (a === 6 && c === 8) return `${base} bottom-[calc(16.666%+8px)] left-0 w-full h-5`      // row 3
    if (a === 0 && c === 6) return `${base} left-[calc(16.666%+8px)] top-0 h-full w-5`          // col 1
    if (a === 1 && c === 7) return `${base} left-1/2 top-0 h-full w-5 -translate-x-1/2`         // col 2
    if (a === 2 && c === 8) return `${base} right-[calc(16.666%+8px)] top-0 h-full w-5`        // col 3
    if (a === 0 && c === 8) return `${base} top-1/2 left-1/2 w-full h-5 rotate-45 origin-center -translate-x-1/2 -translate-y-1/2`
    if (a === 2 && c === 6) return `${base} top-1/2 left-1/2 w-full h-5 -rotate-45 origin-center -translate-x-1/2 -translate-y-1/2`
    return ''
  })
  
  const makeMove = (index: number) => {
    if (board.value[index] !== null || winner.value !== null || currentPlayer.value !== 'X') return
  
    board.value[index] = 'X'
    const { winner: w, combo } = getWinner(board.value)
    if (w !== null) {
      winner.value = w
      winningCombo.value = combo || []
      updateStatusAndStreak(w)
      return
    }
  
    currentPlayer.value = 'O'
    status.value = 'Ход компьютера...'
    nextTick(() => setTimeout(computerMove, 500))
  }
  
  const minimax = (b: (string | null)[], depth: number, isMaximizing: boolean, alpha: number, beta: number): number => {
    const { winner: result } = getWinner(b)
    if (result !== null) {
      return result === 'O' ? 10 - depth : result === 'X' ? depth - 10 : 0
    }
  
    if (isMaximizing) {
      let maxEval = -Infinity
      for (let i = 0; i < 9; i++) {
        if (b[i] === null) {
          b[i] = 'O'
          const evalScore = minimax(b, depth + 1, false, alpha, beta)
          b[i] = null
          maxEval = Math.max(maxEval, evalScore)
          alpha = Math.max(alpha, evalScore)
          if (beta <= alpha) break
        }
      }
      return maxEval
    } else {
      let minEval = Infinity
      for (let i = 0; i < 9; i++) {
        if (b[i] === null) {
          b[i] = 'X'
          const evalScore = minimax(b, depth + 1, true, alpha, beta)
          b[i] = null
          minEval = Math.min(minEval, evalScore)
          beta = Math.min(beta, evalScore)
          if (beta <= alpha) break
        }
      }
      return minEval
    }
  }
  
  const computerMove = () => {
    const emptyCells = board.value.reduce((acc, cell, i) => cell === null ? [...acc, i] : acc, [] as number[])
  
    let move: number | undefined
  
    if (Math.random() < mistakeChance.value) {
      move = emptyCells[Math.floor(Math.random() * emptyCells.length)]
    } else {
      let bestScore = -Infinity
      for (let i of emptyCells) {
        board.value[i] = 'O'
        const score = minimax(board.value, 0, false, -Infinity, Infinity)
        board.value[i] = null
        if (score > bestScore) {
          bestScore = score
          move = i
        }
      }
    }
  
    if (move !== undefined) {
      board.value[move] = 'O'
      const { winner: w, combo } = getWinner(board.value)
      if (w !== null) {
        winner.value = w
        winningCombo.value = combo || []
        updateStatusAndStreak(w)
        return
      }
      currentPlayer.value = 'X'
      status.value = 'Ваш ход (X)'
    }
  }
  
  const updateStatusAndStreak = (w: string) => {
    if (w === 'X') {
      status.value = 'Поздравляем! Вы выиграли!'
      lossStreak.value = 0
    } else if (w === 'O') {
      status.value = 'Компьютер выиграл!'
      lossStreak.value++
    } else {
      status.value = 'Ничья! Попробуйте ещё раз'
      lossStreak.value++
    }
  }
  
  const reset = () => {
    board.value = Array(9).fill(null)
    winner.value = null
    winningCombo.value = []
    currentPlayer.value = 'X'
    status.value = 'Ваш ход (X)'
  }

  const passwordStore = usePasswordStore()
  </script>
  
  <style scoped>
  .line-enter-active {
    transition: all 0.8s cubic-bezier(0.25, 0.8, 0.25, 1);
  }
  .line-enter-from {
    opacity: 0;
    transform: scale(0.5);
  }
  </style>