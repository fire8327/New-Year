<template>
    <div class="min-h-screen flex items-center justify-center bg-[#14120B] text-[#F2F2F2]">
      <div class="flex flex-col items-center gap-12 py-8 px-4 w-full max-w-md md:max-w-lg">
        <h1 class="text-2xl text-center font-black bg-gradient-to-r from-[#F2F2F2] via-[#A8A8A8] to-[#F2F2F2] bg-clip-text text-transparent drop-shadow-2xl">
          Угадай нужную карточку
        </h1>
        
        <div class="text-lg font-semibold text-center px-6 py-2 bg-[#1A170F]/50 backdrop-blur-sm rounded-xl border border-[#333]/50 shadow-xl min-w-[250px]">
          {{ status }}
        </div>
        
        <div class="bg-[#1A170F] rounded-2xl p-8 shadow-2xl shadow-black/50 border border-[#333]/30 w-full max-w-lg">
          <div class="grid grid-cols-3 gap-6 aspect-square">
            <div
              v-for="card in cards"
              :key="card.id"
              class="relative"
            >
              <div
                class="aspect-square rounded-xl flex items-center justify-center text-4xl md:text-5xl font-black cursor-pointer transition-all duration-500 shadow-md border-2 border-transparent"
                :class="cardClasses(card)"
                @click="flipCard(card.id)"
              >
                <span v-if="card.flipped" class="animate-pulse">
                  {{ card.isTarget ? '✓' : '✗' }}
                </span>
                <span v-else class="text-[#A8A8A8]">
                  ?
                </span>
              </div>
            </div>
          </div>
        </div>
        
        <button
          @click="reset"
          class="px-6 py-2 bg-[#F2F2F2]/10 text-[#F2F2F2] font-bold text-lg rounded-2xl hover:bg-[#F2F2F2]/20 active:bg-[#F2F2F2]/5 transition-all duration-300 shadow-lg shadow-black/50 hover:shadow-xl hover:shadow-black/70 border border-[#333]/50 focus:outline-none focus:ring-2 focus:ring-[#A8A8A8]/50"
        >
          {{ gameOver ? 'Играть снова' : 'Новая игра' }}
        </button>
      </div>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, computed, onMounted } from 'vue'
  
  interface Card {
    id: number
    isTarget: boolean
    flipped: boolean
  }
  
  const cards = ref<Card[]>([])
  const status = ref<string>('Выберите карточку')
  const gameOver = ref<boolean>(false)
  
  // Текущее количество карточек (начинаем с 9)
  const currentCount = ref<number>(9)
  
  const cardClasses = (card: Card) => {
    if (!card.flipped) {
      return 'bg-[#0F0D08]/80 hover:bg-[#201B10]/80 hover:scale-105 active:scale-95'
    }
    
    if (card.isTarget) {
      return 'bg-[#00FF88]/20 shadow-[0_0_30px_#00FF88_/_0.6] border-[#00FF88]/40'
    }
    
    return 'bg-[#FF0088]/20 shadow-[0_0_20px_#FF0088_/_0.4] border-[#FF0088]/30'
  }
  
  const initializeGame = () => {
    const targetIndex = Math.floor(Math.random() * currentCount.value)
  
    cards.value = Array.from({ length: currentCount.value }, (_, i) => ({
      id: i,
      isTarget: i === targetIndex,
      flipped: false
    }))
  
    status.value = `Найдите правильную карточку (1 из ${currentCount.value})`
    gameOver.value = false
  }
  
  const flipCard = (id: number) => {
    if (gameOver.value) return
  
    const card = cards.value.find(c => c.id === id)
    if (!card || card.flipped) return
  
    // Переворачиваем выбранную
    card.flipped = true
  
    // Переворачиваем все остальные для показа результата
    cards.value.forEach(c => { c.flipped = true })
  
    gameOver.value = true
  
    if (card.isTarget) {
      status.value = 'Поздравляем! Вы угадали!'
      currentCount.value = 9 // Сброс до максимума после победы
    } else {
      status.value = 'Не угадали. В следующий раз будет легче...'
      currentCount.value = Math.max(1, currentCount.value - 1) // Уменьшаем на 1, минимум 1
    }
  }
  
  const reset = () => {
    initializeGame()
  }
  
  onMounted(() => {
    initializeGame()
  })
  </script>