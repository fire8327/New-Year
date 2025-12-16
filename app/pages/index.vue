<template>
    <div v-if="!passwordStore.password || (passwordStore.password != 1 && passwordStore.password != 2)" class="grow flex items-center justify-center">
        <div class="w-[calc(100%-16px)] md:w-1/2 lg:w-1/3 p-4 border border-white/10 rounded-xl flex flex-col gap-6">
            <p class="text-xl font-mono tracking-wide font-medium text-center">Введите пароль</p>
            <FormKit v-model="passwordStore.password" type="password" outer-class="w-full" input-class="w-full px-4 py-1.5 rounded-xl text-sm font-medium bg-transparent border border-white/15 focus:border-white/50 focus:outline-none" placeholder="Пароль"/>
        </div>
    </div>
    <TicTacToe v-if="passwordStore.password == 1"></TicTacToe>
    <CardToe v-if="passwordStore.password == 2"></CardToe>
</template>

<script setup>
/* форма паролей */
const passwordStore = usePasswordStore()
</script>