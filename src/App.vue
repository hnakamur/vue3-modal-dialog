<template>
  <button class="btn" @click="onClick">Open Dialog</button>
  <div class="padding"></div>
  <p>hello</p>
  <MyDialog ref="myDialog" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MyDialog from './components/MyDialog.vue'

const myDialog = ref<typeof MyDialog | null>(null)

async function onClick() {
  const result = await myDialog.value?.openDialog()
  console.log('result=', result)
}
</script>

<style scoped>
.padding {
  height: 100vh;
  border: 1px solid black;
}
</style>
