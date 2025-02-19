<template>
  <div
    data-partykit="generative-preview-card"
    class="border bg-background-color shadow-primary p-5 pb-6 w-full h-min lg:max-w-[490px]">
    <BaseMediaItem
      :src="sanitizeIpfsUrl(displayUrl)"
      :mime-type="generativeImageUrl ? 'text/html' : ''"
      preview
      is-detail
      class="border" />

    <NeoButton
      v-if="isLoading"
      class="mt-5 h-[40px] border-k-grey pointer-events-auto cursor-wait hover:!bg-transparent"
      expanded
      rounded
      no-shadow
      disabled>
      <div class="inline-flex items-center">
        <span class="mr-2">{{ $t('mint.unlockable.generating') }}</span>
        <NeoIcon icon="circle-notch" spin />
      </div>
    </NeoButton>

    <NeoButton
      v-else
      class="mt-5 h-[40px] border-k-grey hover:!bg-transparent"
      expanded
      rounded
      no-shadow
      @click="generateNft()">
      <div class="inline-flex items-center">
        <span>{{ $t('drops.createNewVariation') }}</span>
        <NeoIcon icon="arrow-rotate-left" class="ml-2" />
      </div>
    </NeoButton>

    <hr class="my-5" />

    <div class="flex justify-between items-center mb-4">
      <div class="font-bold">
        <span v-if="!!Number(drop.price)">{{ formattedPrice }}</span>
        <span v-else>{{ $t('free') }}</span>
      </div>
      <div class="flex justify-end items-center">
        <div class="mr-4 text-neutral-7">{{ mintedPercent }}% ~</div>
        <div class="font-bold">
          {{ mintedCount }}/{{ maxCount }}
          {{ $t('statsOverview.minted') }}
        </div>
      </div>
    </div>

    <CollectionUnlockableSlider
      class="text-neutral-5 dark:text-neutral-9"
      :value="mintedCount / maxCount" />

    <CollectionDropMintButton
      class="mt-6"
      :collection-id="collectionId"
      :user-minted-nft-id="userMintedNftId"
      :is-wallet-connecting="isWalletConnecting"
      :is-image-fetching="isImageFetching"
      :is-loading="isLoading"
      :minimum-funds="minimumFunds"
      :max-count="maxCount"
      :mint-count-available="mintCountAvailable"
      :mint-button="mintButton"
      :holder-of-collection="holderOfCollection"
      @mint="emit('mint')" />
  </div>
</template>

<script setup lang="ts">
import { blake2AsHex, encodeAddress } from '@polkadot/util-crypto'
import { NeoButton, NeoIcon } from '@kodadot1/brick'
import { sanitizeIpfsUrl } from '@/utils/ipfs'
import { DropItem } from '@/params/types'
import type {
  HolderOfCollectionProp,
  MinimumFundsProp,
  MintButtonProp,
} from '@/components/collection/drop/types'
import { getRandomIntFromRange } from '../unlockable/utils'

const props = defineProps<{
  drop: DropItem
  minted: number
  collectionId: string
  mintedCount: number
  mintCountAvailable: boolean
  maxCount: number
  minimumFunds: MinimumFundsProp
  isImageFetching: boolean
  isWalletConnecting: boolean
  isLoading: boolean
  mintButton: MintButtonProp
  userMintedNftId?: string
  holderOfCollection?: HolderOfCollectionProp
}>()

const emit = defineEmits(['select', 'mint'])

const { accountId } = useAuth()
const { chainSymbol, decimals } = useChain()

const mintedPercent = computed(() => {
  const percent = (props.mintedCount / props.maxCount) * 100
  return Math.round(percent)
})

const { formatted: formattedPrice } = useAmount(
  computed(() => props.drop.price),
  decimals,
  chainSymbol,
)

const STEP = 64
const entropyRange = computed<[number, number]>(() => [
  STEP * props.minted,
  STEP * (props.minted + 1),
])

const getHash = (isDefault?: boolean) => {
  const ss58Format = isDefault
    ? entropyRange.value[0]
    : getRandomIntFromRange(entropyRange.value[0], entropyRange.value[1])

  // https://github.com/paritytech/ss58-registry/blob/30889d6c9d332953a6e3333b30513eef89003f64/ss58-registry.json#L1292C17-L1292C22
  const initialValue = accountId.value
    ? encodeAddress(accountId.value, ss58Format)
    : String(Date.now() << ss58Format)
  return blake2AsHex(initialValue, 256, null, true)
}

const generativeImageUrl = ref(
  accountId.value ? `${props.drop.content}/?hash=${getHash(true)}` : '',
)

const isLoading = ref(false)

const displayUrl = computed(() => {
  return generativeImageUrl.value || props.drop.image
})
const generateNft = (isDefault: boolean = false) => {
  isLoading.value = true
  const metadata = `${props.drop.content}/?hash=${getHash(isDefault)}`
  generativeImageUrl.value = metadata
  emit('select', generativeImageUrl.value)

  setTimeout(() => {
    isLoading.value = false
  }, 3000)
}

watch(
  accountId,
  () => {
    generateNft(true)
  },
  {
    immediate: true,
  },
)
</script>
