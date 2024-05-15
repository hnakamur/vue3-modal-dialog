<template>
  <dialog ref="dialog" class="modal">
    <div v-if="open" class="modal-box">
      <h3 class="font-bold text-lg">Hello!</h3>
      <p><a href="https://daisyui.com/components/modal/">Daisy UI Modal Dialog example</a></p>
      <p class="py-4">Press ESC key or click the button below to close</p>
      <div class="modal-action">
        <!-- if there is a button in form, it will close the modal -->
        <button class="btn btn-secondary" @click="onDialogAction('cancel')">Cancel</button>
        <button class="btn btn-primary" @click="onDialogAction('close')">Close</button>
      </div>
    </div>
  </dialog>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick } from 'vue'

// APIを公開
defineExpose({
  openDialog
})

// dialogの参照を保持する変数
const dialog = ref<HTMLDialogElement | null>(null)
const open = ref(false)

const firstElement = ref<HTMLElement | undefined>()
const lastElement = ref<HTMLElement | undefined>()

// Promiseのresolveをキャッシュ
let resolve: (action: 'cancel' | 'close') => void

const focusableCandidateSelectors =
  'a[href], button, textarea, input[type="text"], input[type="email"], input[type="radio"], input[type="checkbox"], select'

// 外部に公開するAPI。ダイアログを開いて、終了
async function openDialog() {
  open.value = true
  dialog.value?.showModal()
  nextTick(() => {
    const focusableElements = dialog.value?.querySelectorAll(focusableCandidateSelectors) as
      | NodeListOf<HTMLElement>
      | undefined
    firstElement.value = focusableElements?.[0]
    lastElement.value = focusableElements?.[focusableElements.length - 1]
    firstElement.value?.focus()
  })
  const promise = new Promise<'cancel' | 'close'>((res) => {
    resolve = res
  })

  const result = await promise
  dialog.value?.close()
  open.value = false
  return result
}

// ダイアログ側の閉じるボタンが押されたときに呼ばれるコールバック
function onDialogAction(action: 'cancel' | 'close') {
  resolve(action)
}

// escapeキーで閉じるのをフックして、実装しようとする終了と同じ流れに載せる
function handleEscape(event: Event) {
  event.preventDefault()
  resolve('cancel')
}

function handleKeydown(e: KeyboardEvent) {
  if (e.key === 'Tab') {
    if (e.shiftKey) {
      if (document.activeElement === firstElement.value) {
        e.preventDefault()
        lastElement.value?.focus()
      }
    } else {
      if (document.activeElement === lastElement.value) {
        e.preventDefault()
        firstElement.value?.focus()
      }
    }
  }
}

onMounted(() => {
  dialog.value?.addEventListener('cancel', handleEscape)
  dialog.value?.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  dialog.value?.removeEventListener('keydown', handleKeydown)
  dialog.value?.removeEventListener('cancel', handleEscape)
})
</script>
