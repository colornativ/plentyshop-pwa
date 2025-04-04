<template>
  <UiBlockPlaceholder v-if="displayTopPlaceholder(index)" />
  <div
    :class="[
      'relative group',
      {
        'mb-s': blockSize === 's',
        'mb-m': blockSize === 'm',
        'mb-l': blockSize === 'l',
        'mb-xl': blockSize === 'xl',
      },
      {
        'max-w-screen-3xl mx-auto lg:px-10 mt-3': block.name !== 'BannerCarousel',
      },
      {
        'px-4 md:px-6': block.name !== 'BannerCarousel' && block.name !== 'NewsletterSubscribe',
      },
      {
        'outline outline-4 outline-[#538AEA]':
          isPreview && disableActions && isClicked && isTablet && clickedBlockIndex === index,
      },
      { 'hover:outline hover:outline-4 hover:outline-[#538AEA]': isPreview && disableActions && !isTablet },
    ]"
    data-testid="block-wrapper"
    @click="tabletEdit(index)"
  >
    <button
      v-if="disableActions && isPreview"
      class="z-[0] md:z-[1] lg:z-[10] absolute top-0 left-1/2 transform -translate-x-1/2 -translate-y-1/2 rounded-[18px] p-[6px] bg-[#538aea] text-white opacity-0 hover:opacity-100 group-hover:opacity-100 group-focus:opacity-100"
      :class="[{ 'opacity-100': isClicked && clickedBlockIndex === index }]"
      data-testid="top-add-block"
      aria-label="top add block"
      @click.stop="addNewBlock(index, -1)"
    >
      <SfIconAdd class="cursor-pointer" />
    </button>
    <UiBlockActions
      v-if="disableActions && blockHasData && blockHasData(block) && isPreview"
      :class="[
        'opacity-0',
        {
          'hover:opacity-100 group-hover:opacity-100 group-focus:opacity-100': !isTablet,
          'opacity-100': isTablet && isClicked && clickedBlockIndex === index,
        },
      ]"
      :index="index"
      :blocks="block"
      :is-last-block="isLastBlock(index)"
      @delete="deleteBlock"
      @change-position="changeBlockPosition"
    />
    <component :is="getBlockComponent" v-bind="block.options" :index="index" />
    <button
      v-if="disableActions && isPreview"
      class="z-[0] md:z-[1] lg:z-[10] absolute bottom-0 left-1/2 transform -translate-x-1/2 translate-y-1/2 rounded-[18px] p-[6px] bg-[#538aea] text-white opacity-0 group-hover:opacity-100 group-focus:opacity-100"
      :class="[{ 'opacity-100': isClicked && clickedBlockIndex === index }]"
      data-testid="bottom-add-block"
      aria-label="bottom add block"
      @click.stop="addNewBlock(index, 1)"
    >
      <SfIconAdd class="cursor-pointer" />
    </button>
  </div>
  <UiBlockPlaceholder v-if="displayBottomPlaceholder(index)" />
</template>

<script lang="ts" setup>
import { SfIconAdd } from '@storefront-ui/vue';

interface Props {
  index: number;
  block: Block;
  isPreview: boolean;
  disableActions: boolean;
  isClicked: boolean;
  clickedBlockIndex: number | null;
  isTablet: boolean;
  blockHasData?: (block: Block) => boolean;
  tabletEdit: (index: number) => void;
  addNewBlock: (index: number, position: number) => void;
  changeBlockPosition: (index: number, position: number) => void;
  isLastBlock: (index: number) => boolean;
  deleteBlock: (index: number) => void;
}

const props = defineProps<Props>();

const { blockSize, drawerOpen, drawerView } = useSiteConfiguration();
const { visiblePlaceholder } = useBlockManager();

const modules = import.meta.glob('@/components/**/blocks/**/*.vue') as Record<
  string,
  () => Promise<{ default: unknown }>
>;
const getBlockComponent = computed(() => {
  if (!props.block.name) return null;

  const regex = new RegExp(`${props.block.name}\\.vue$`, 'i');
  const matched = Object.keys(modules).find((path) => regex.test(path));

  if (matched) {
    return defineAsyncComponent({
      loader: modules[matched],
    });
  }

  return '';
});

const displayTopPlaceholder = (index: number): boolean => {
  const visiblePlaceholderState = visiblePlaceholder.value;

  return (
    visiblePlaceholderState.position === 'top' &&
    visiblePlaceholderState.index === index &&
    drawerOpen.value &&
    drawerView.value === 'blocksList'
  );
};

const displayBottomPlaceholder = (index: number): boolean => {
  const visiblePlaceholderState = visiblePlaceholder.value;

  return (
    visiblePlaceholderState.position === 'bottom' &&
    visiblePlaceholderState.index === index &&
    drawerOpen.value &&
    drawerView.value === 'blocksList'
  );
};
</script>
