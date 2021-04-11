<template>
  <main class="p-6 flex">
    <div class="relative grid grid-cols-5 grid-rows-4 gap-3 mr-auto">
      <!-- Elements for spacing purposes only -->
      <div
        v-for="id in elements"
        :key="id"
        class="h-24 w-24 flex items-center justify-center text-2xl bg-red-400 text-transparent pointer-events-none"
        aria-hidden="true"
      ></div>
      <!-- Elements that will be dragged and dropped -->
      <div
        v-for="(id, index) in elements"
        :key="id"
        :ref="dragDropSwap.elements.getRef(index)"
        class="absolute h-24 w-24 flex items-center justify-center text-2xl bg-blue-300 text-blue-900"
        :style="{
          top: `calc(0.75rem * ${Math.floor((id - 1) / 5)} + 6rem * ${Math.floor((id - 1) / 5)})`,
          left: `calc(0.75rem * ${(id - 1) % 5} + 6rem * ${(id - 1) % 5})`,
        }"
      >
        {{ id }}
      </div>
    </div>
  </main>
</template>

<script>
import { ref, readonly, onMounted, onUpdated, onBeforeUpdate } from 'vue'
import { useListenable } from '@baleada/vue-composition'
import { mousedragdrop } from '@baleada/recognizeable-handlers'
import { on } from '@baleada/vue-features'

export default {
  name: 'App',
  setup () {
    const elements = new Array(20).fill().map((el, id) => id + 1),
          targetsApi = useTargets(),
          dragDropSwap = readonly({
            elements: targetsApi
          })

    on({
      target: targetsApi.els,
      events: {
        recognizeable: {
          targetClosure: () => () => { /* Do nothing. Logic is handled in mousedragdrop hooks */ },
          options: {
            listenable: {
              recognizeable: {
                handlers: mousedragdrop({
                  onMove: ({ event: { target }, metadata: { distance: { horizontal, vertical } } }) => {
                    const id = Number(target.textContent)

                    target.style.top = `calc(0.75rem * ${Math.floor((id - 1) / 5)} + 6rem * ${Math.floor((id - 1) / 5)} + ${vertical.fromStart}px)`
                    target.style.left = `calc(0.75rem * ${(id - 1) % 5} + 6rem * ${(id - 1) % 5} + ${horizontal.fromStart}px)`
                  }
                }),
              },
            },
          }
        }
      }
    })


    const listenable = useListenable('recognizeable', {
      recognizeable: {
        handlers: mousedragdrop()
      }
    })

    return {
      elements,
      dragDropSwap,
    }
  }
}

function useTargets (effect) {
  const targets = ref([]),
        handle = index => target => {
          if (target) targets.value[index] = target
        },
        api = {
          getRef: handle,
          els: targets
        }

  onBeforeUpdate(() => (targets.value = []))
      
  onMounted(() => effect?.())
  onUpdated(() => effect?.())

  return api
}

function createSwap ({ from, to }) {
  return array => {
    const ensuredFrom = from < to ? from : to,
          ensuredTo = from < to ? to : from
    
    return [
      ...array.slice(0, ensuredFrom),
      array[ensuredTo],
      ...array.slice(ensuredFrom + 1, ensuredTo),
      array[ensuredFrom],
      ...array.slice(ensuredTo)
    ]
  }
}
</script>
