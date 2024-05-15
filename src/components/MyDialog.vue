<template>
  <dialog ref="dialog" class="modal">
    <!-- ダイアログをdivタグで作る場合はpreTrapとpostTrapはダイアログの外で良いが、
         dialogタグで作る場合は外だとフォーカスが来なかったので内側に置く -->
    <div tabindex="0" ref="preTrap" />
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
    <div tabindex="0" ref="postTrap" />
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

// Promiseのresolveをキャッシュ
let resolve: (action: 'cancel' | 'close') => void

const preTrap = ref<HTMLElement>()
const postTrap = ref<HTMLElement>()
const ignoresFocusChange = ref(false)
const lastFocus = ref<Element | null>(null)

// focusableCandidateSelectors は以下の2つを参考にしました。
// https://github.com/KittyGiraudel/focusable-selectors/blob/799829e3b8c329d679b3b414b5dfcfa257a817cf/index.js
// https://github.com/focus-trap/tabbable/blob/baa8c3044fe0a8fd8c0826f4a3e284872e1467a5/src/index.js#L1-L13
const focusableCandidateSelectors =
  'a[href]:not([tabindex^="-"])' +
  ',area[href]:not([tabindex^="-"])' +
  ',input:not([type="hidden"]):not([type="radio"]):not([disabled]):not([tabindex^="-"])' +
  ',input[type="radio"]:not([disabled]):not([tabindex^="-"])' +
  ',select:not([disabled]):not([tabindex^="-"])' +
  ',textarea:not([disabled]):not([tabindex^="-"])' +
  ',button:not([disabled]):not([tabindex^="-"])' +
  ',iframe:not([tabindex^="-"])' +
  ',audio[controls]:not([tabindex^="-"])' +
  ',video[controls]:not([tabindex^="-"])' +
  ',[contenteditable]:not([tabindex^="-"])' +
  ',[tabindex]:not([tabindex^="-"])' +
  ',details>summary:not([tabindex^="-"])' +
  ',details:not([tabindex^="-"])'

// 外部に公開するAPI。ダイアログを開いて、終了
async function openDialog() {
  open.value = true
  dialog.value?.showModal()
  nextTick(() => {
    focusFirstDescendant(dialog.value)
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

// attemptFocus, focusFirstDescendant, focusLastDesendant, trapFocus のロジックは
// https://www.w3.org/TR/wai-aria-practices-1.1/examples/dialog-modal/dialog.html
// を参考にしました。

const attemptFocus = (element: HTMLElement) => {
  ignoresFocusChange.value = true
  try {
    element.focus()
  } catch (e) {
    console.warn('focus failed', e)
  }
  ignoresFocusChange.value = false
  return element === document.activeElement
}

const focusFirstDescendant = (element: HTMLElement | null) => {
  if (element) {
    const descendants = element.querySelectorAll<HTMLElement>(focusableCandidateSelectors)
    for (const el of descendants) {
      if (el === preTrap.value) continue
      if (attemptFocus(el)) break
    }
  }
}

const focusLastDescendant = (element: HTMLElement | null) => {
  if (element) {
    const descendants = element.querySelectorAll<HTMLElement>(focusableCandidateSelectors)
    for (let i = descendants.length - 1; i >= 0; i--) {
      if (descendants[i] === postTrap.value) continue
      if (attemptFocus(descendants[i])) break
    }
  }
}

const trapFocus = (event: FocusEvent) => {
  if (!ignoresFocusChange.value && dialog.value != null) {
    const dialogNode = dialog.value as HTMLElement
    const target = event.target as HTMLElement
    if (dialogNode.contains(target) && target !== preTrap.value && target !== postTrap.value) {
      lastFocus.value = target
    } else {
      focusFirstDescendant(dialogNode)
      if (lastFocus.value === document.activeElement) {
        focusLastDescendant(dialogNode)
      }
      lastFocus.value = document.activeElement
    }
  }
}

onMounted(() => {
  dialog.value?.addEventListener('cancel', handleEscape)
  dialog.value?.addEventListener('focus', trapFocus, true)
})

onUnmounted(() => {
  dialog.value?.removeEventListener('focus', trapFocus)
  dialog.value?.removeEventListener('cancel', handleEscape)
})
</script>
