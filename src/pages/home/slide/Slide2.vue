<template>
  <SlideItem class="slide-item-class">
    <SlideList
      :active="props.active"
      uniqueId="follow"
      :style="{
        background: 'black'
      }"
      :api="recommendedVideo"
    />
  </SlideItem>
</template>

<script setup lang="jsx">
import SlideItem from '@/components/slide/SlideItem.vue'
import SlideList from './SlideList.vue'
import { recommendedVideo } from '@/api/videos'

const props = defineProps({
  active: {
    type: Boolean,
    default: false
  }
})
</script>

<style scoped lang="less">
.slide-item-class {
  position: relative;
}
</style>
