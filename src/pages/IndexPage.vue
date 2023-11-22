<script  setup lang="ts">
import { useMouse, useWindowSize } from '@vueuse/core'
import { computed } from 'vue'
import GlTriangle from 'src/components/GlTriangle.vue'

const { x, y } = useMouse()
const { width, height } = useWindowSize()

const dx = computed(() => x.value - width.value / 2)
const dy = computed(() => y.value - height.value / 2)
const distance = computed(() => Math.sqrt(dx.value ** 2 + dy.value ** 2))
const size = computed(() => Math.max(250 - distance.value / 3, 100))
const opacity = computed(() => 100 / size.value)
const blur = computed(() => Math.max(64 * (1 - opacity.value), 18))

const cardGradient = computed(() => {
  const xPos = (x.value / width.value) * 100
  const yPos = (y.value / height.value) * 100

  return `radial-gradient(circle at ${xPos}% ${yPos}%, rgba(0, 0, 0, 1), transparent)`
})
</script>

<template>
  <q-page class="row items-center justify-evenly q-pa-lg gradient">
    <q-card :style="{ maskImage: cardGradient }">
      <q-card-section>
        <GlTriangle />
      </q-card-section>
    </q-card>
  </q-page>
  <div
    class="q-pa-lg absolute circle-holder"
    :style="{ top: `${y}px`, left: `${x}px`, opacity: opacity ** 2, filter: `blur(${blur}px)` }"
  >
    <div class="text-center">
      <div class="bg-white round q-pa-lg" :style="{ width: `${size}px`, height: `${size}px` }" />
    </div>
  </div>
</template>

<style lang="scss" scoped>
.gradient {
  background: linear-gradient(to top, #add8e6, $primary);
}
.round {
  border-radius: 50%;
}
.circle-holder {
  transform: translate(-50%, -50%);
  pointer-events: none;
}
</style>
