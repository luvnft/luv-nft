<template>
  <div>
    <CarouselModuleCarouselAgnostic
      v-if="isReady"
      :key="dropsAlias.join('-')"
      v-slot="{ item }"
      :items="drops"
      :step="steps"
      :breakpoints="breakpoints">
      <DropsDropCard :drop="item" />
    </CarouselModuleCarouselAgnostic>
    <CarouselModuleCarouselAgnostic
      v-else
      :items="Array(DROP_SKELETON_COUNT).fill({ id: 'drop-skeleton' })"
      :step="steps"
      :breakpoints="breakpoints">
      <DropsDropCardSkeleton />
    </CarouselModuleCarouselAgnostic>
  </div>
</template>

<script lang="ts" setup>
import { type CarouseBreakpointsConfig } from '@/components/carousel/module/CarouselAgnostic.vue'
import { useDrops } from '@/components/drops/useDrops'

const DROP_SKELETON_COUNT = 3

const breakpoints: CarouseBreakpointsConfig = {
  '640px': { slides: { perView: 1.2, spacing: 16 } },
  '768px': {
    slides: { perView: 1.2, spacing: 16 },
  },
  '1024px': {
    slides: { perView: 2.2, spacing: 16 },
  },
  '1280px': {
    slides: { perView: 3, spacing: 32 },
  },
  '1540px': {
    slides: { perView: 3, spacing: 32 },
  },
}

let queries = {
  limit: 6,
  active: [true],
  chain: ['ahp'],
}

if (!isProduction) {
  queries = {
    limit: 12,
    active: [true, false],
    chain: ['ahp', 'ahk'],
  }
}

const { drops, loaded: isReady } = useDrops(queries)
const dropsAlias = computed(() => drops.value.map((drop) => drop.alias))
const { width } = useWindowSize()

const steps = computed(() => {
  if (width.value > 1024) {
    return 3
  }
  if (width.value > 768) {
    return 2
  }
  return 1
})
</script>
