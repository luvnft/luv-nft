<template>
  <div
    class="nft-media-info flex flex-col"
    :class="`nft-media-info__${variant}`">
    <div class="flex flex-col">
      <span
        class="is-ellipsis font-bold"
        data-testid="nft-name"
        :title="name"
        >{{ name || '--' }}</span
      >
      <CollectionDetailsPopover
        v-if="!isMinimal && (nft.collection.name || nft.collection.id)"
        :show-delay="collectionPopoverShowDelay"
        class="text-xs text-k-grey hover:text-text-color is-ellipsis"
        :nft="nft">
        <template #content>
          <a
            :v-safe-href="`/${prefix}/collection/${nft.collection.id}`"
            class="text-k-grey hover:text-text-color">
            {{ nft.collection.name || '--' }}
          </a>
        </template>
      </CollectionDetailsPopover>
    </div>

    <div
      class="flex items-center mt-2 is-ellipsis nft-media-info-footer"
      :class="[showPrice ? 'justify-between' : 'justify-end']">
      <CommonTokenMoney
        v-if="showPrice"
        :value="nft.price ?? nft.cheapest?.price"
        data-testid="card-money" />
      <span v-if="!isMinimal" class="text-k-grey capitalize text-xs">{{
        getChainNameByPrefix(prefix)
      }}</span>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed } from 'vue'

import CommonTokenMoney from '@/components/shared/CommonTokenMoney.vue'
import { getChainNameByPrefix } from '@/utils/chain'
import type { NeoNFT, NftCardVariant } from './types'
import { addSnSuffixName } from '@/utils/nft'

const props = withDefaults(
  defineProps<{
    nft: NeoNFT
    prefix: string
    showPrice?: boolean
    collectionPopoverShowDelay?: number
    variant?: NftCardVariant
    displayNameWithSn?: boolean
  }>(),
  {
    collectionPopoverShowDelay: 500,
    variant: 'primary',
  },
)

const name = computed(() => {
  const originalName = props.nft.name
  if (!props.displayNameWithSn) {
    return originalName
  }
  const sn = isTokenEntity(props.nft)
    ? props.nft?.cheapest?.id?.split('-')[1]
    : props.nft?.sn

  return sn ? addSnSuffixName(props.nft.name, sn) : originalName
})
const isMinimal = computed(() =>
  props.variant ? props.variant.includes('minimal') : false,
)
</script>
