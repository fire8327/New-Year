<template>
    <div class="h-screen flex items-center justify-center bg-[#14120B] text-[#F2F2F2]">
      <div class="flex flex-col items-center gap-12 py-8 px-4 w-full max-w-md md:max-w-lg">
        <h1 class="text-2xl font-black bg-gradient-to-r from-[#F2F2F2] via-[#A8A8A8] to-[#F2F2F2] bg-clip-text text-transparent drop-shadow-2xl">
          Камень ✊ Ножницы ✌️ Бумага ✋
        </h1>
        
        <div class="text-lg font-semibold text-center px-6 py-2 bg-[#1A170F]/50 backdrop-blur-sm rounded-xl border border-[#333]/50 shadow-xl min-w-[250px]">
          {{ status }}
        </div>
        
        <div class="flex flex-col items-center gap-8">
          <!-- Счёт -->
          <div class="flex gap-8 text-3xl md:text-4xl font-black">
            <div class="text-[#00FF88] drop-shadow-2xl">{{ playerWins }}</div>
            <div class="text-[#FF0088] drop-shadow-2xl">VS</div>
            <div class="text-[#FF0088] drop-shadow-2xl">{{ cpuWins }}</div>
          </div>
          
          <!-- Результат раунда -->
          <div v-if="roundResult" class="text-6xl md:text-7xl animate-bounce">
            {{ roundResult.emoji }}
          </div>
          
          <!-- Анимация выбора -->
          <transition name="fade">
            <div v-if="showChoices" class="flex gap-12 md:gap-20 text-6xl md:text-7xl">
              <span :class="playerChoiceClass">{{ playerChoice?.emoji }}</span>
              <span class="text-[#A8A8A8]">🤝</span>
              <span :class="cpuChoiceClass">{{ cpuChoice?.emoji }}</span>
            </div>
          </transition>
        </div>
        
        <!-- Кнопки выбора (только когда можно играть) -->
        <div v-if="!gameOver && !showChoices" class="flex gap-4 md:gap-6">
          <button
            @click="play('rock')"
            class="flex flex-col items-center gap-2 p-4 bg-[#0F0D08]/80 rounded-2xl hover:bg-[#201B10]/80 hover:scale-110 active:scale-95 transition-all duration-300 shadow-lg border border-[#333]/50 hover:shadow-[0_0_20px_#00FF88_/_0.4]"
          >
            <span class="text-4xl">✊</span>
            <span class="text-sm font-bold">Камень</span>
          </button>
          <button
            @click="play('scissors')"
            class="flex flex-col items-center gap-2 p-4 bg-[#0F0D08]/80 rounded-2xl hover:bg-[#201B10]/80 hover:scale-110 active:scale-95 transition-all duration-300 shadow-lg border border-[#333]/50 hover:shadow-[0_0_20px_#FF0088_/_0.4]"
          >
            <span class="text-4xl">✌️</span>
            <span class="text-sm font-bold">Ножницы</span>
          </button>
          <button
            @click="play('paper')"
            class="flex flex-col items-center gap-2 p-4 bg-[#0F0D08]/80 rounded-2xl hover:bg-[#201B10]/80 hover:scale-110 active:scale-95 transition-all duration-300 shadow-lg border border-[#333]/50 hover:shadow-[0_0_20px_#A8A8A8_/_0.4]"
          >
            <span class="text-4xl">✋</span>
            <span class="text-sm font-bold">Бумага</span>
          </button>
        </div>
        
        <div class="flex items-center gap-2">
            <button
              v-if="gameOver"
              @click="reset"
              class="px-6 py-2 bg-[#F2F2F2]/10 text-[#F2F2F2] font-bold text-lg rounded-2xl hover:bg-[#F2F2F2]/20 active:bg-[#F2F2F2]/5 transition-all duration-300 shadow-lg shadow-black/50 hover:shadow-xl hover:shadow-black/70 border border-[#333]/50 focus:outline-none focus:ring-2 focus:ring-[#A8A8A8]/50"
            >
              Играть снова
            </button>
            <button
              @click="passwordStore.password = ''"
              class="px-6 py-2 bg-[#F2F2F2]/10 text-[#F2F2F2] font-bold text-lg rounded-2xl hover:bg-[#F2F2F2]/20 active:bg-[#F2F2F2]/5 transition-all duration-300 shadow-lg shadow-black/50 hover:shadow-xl hover:shadow-black/70 border border-[#333]/50 focus:outline-none focus:ring-2 focus:ring-[#A8A8A8]/50"
            >
              В меню
            </button>
        </div>
      </div>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, computed, nextTick } from 'vue'
  
  interface Choice {
    name: 'rock' | 'scissors' | 'paper'
    emoji: string
    beats: Choice['name'][]
  }
  
  const choices: Record<Choice['name'], Choice> = {
    rock: { name: 'rock', emoji: '✊', beats: ['scissors'] },
    scissors: { name: 'scissors', emoji: '✌️', beats: ['paper'] },
    paper: { name: 'paper', emoji: '✋', beats: ['rock'] }
  }
  
  const playerWins = ref(0)
  const cpuWins = ref(0)
  const lossStreak = ref(0)
  const gameOver = ref(false)
  const showChoices = ref(false)
  const roundResult = ref<{ winner: 'player' | 'cpu' | 'draw', emoji: string } | null>(null)
  const playerChoice = ref<Choice | null>(null)
  const cpuChoice = ref<Choice | null>(null)
  const status = ref('Выберите свой ход')
  
  const mistakeChance = computed(() => Math.min(lossStreak.value * 0.2, 0.6)) // До 60% шанс ошибки компа
  
  const playerChoiceClass = computed(() => ({
    'text-[#00FF88] shadow-[0_0_30px_#00FF88_/_0.6] animate-pulse': playerChoice.value,
    'text-[#FF0088] shadow-[0_0_30px_#FF0088_/_0.6] animate-pulse': cpuChoice.value
  }))
  
  const cpuChoiceClass = computed(() => ({
    'text-[#FF0088] shadow-[0_0_30px_#FF0088_/_0.6] animate-pulse': cpuChoice.value
  }))
  
  const play = (playerMove: Choice['name']) => {
    playerChoice.value = choices[playerMove]
    showChoices.value = true
    status.value = 'Компьютер думает...'
  
    nextTick(() => setTimeout(computerPlay, 1500))
  }
  
  const computerPlay = () => {
    let cpuMove: Choice['name']
  
    // С шансом ошибки - комп "дарит" проигрышный ход
    if (Math.random() < mistakeChance.value) {
      // Выбираем ход, который проигрывает игроку
      cpuMove = playerChoice.value!.beats[0] as Choice['name'] // Всегда один beats
    } else {
      // Нормальный рандом
      const keys = Object.keys(choices) as Choice['name'][]
      cpuMove = keys[Math.floor(Math.random() * 3)]
    }
  
    cpuChoice.value = choices[cpuMove]
  
    // Определяем победителя
    const result = getWinner(playerChoice.value!, cpuChoice.value!)
    roundResult.value = result
  
    if (result.winner === 'player') {
      playerWins.value++
      status.value = 'Вы выиграли раунд!'
      lossStreak.value = 0
    } else if (result.winner === 'cpu') {
      cpuWins.value++
      status.value = 'Компьютер выиграл раунд!'
      lossStreak.value++
    } else {
      status.value = 'Ничья!'
    }
  
    // Проверяем конец матча (до 3 побед)
    if (playerWins.value >= 3 || cpuWins.value >= 3) {
      gameOver.value = true
      if (playerWins.value >= 3) {
        status.value = 'Локация: в зале под кроватью'
      } else {
        status.value = 'Компьютер выиграл матч. Попробуйте снова!'
      }
    } else {
      // Новый раунд через 2 сек
      setTimeout(() => {
        resetRound()
      }, 2000)
    }
  }
  
  const getWinner = (player: Choice, cpu: Choice): { winner: 'player' | 'cpu' | 'draw', emoji: string } => {
    if (player.beats.includes(cpu.name)) {
      return { winner: 'player' as const, emoji: '✅' }
    } else if (cpu.beats.includes(player.name)) {
      return { winner: 'cpu' as const, emoji: '❌' }
    }
    return { winner: 'draw' as const, emoji: '🤝' }
  }
  
  const resetRound = () => {
    playerChoice.value = null
    cpuChoice.value = null
    roundResult.value = null
    showChoices.value = false
    status.value = 'Выберите свой ход'
  }
  
  const reset = () => {
    playerWins.value = 0
    cpuWins.value = 0
    lossStreak.value = 0
    gameOver.value = false
    resetRound()
  }

  const passwordStore = usePasswordStore()
  </script>
  
  <style scoped>
  .fade-enter-active, .fade-leave-active {
    transition: opacity 0.5s;
  }
  .fade-enter-from, .fade-leave-to {
    opacity: 0;
  }
  </style>