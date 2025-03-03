<script setup lang="ts">
import { useLayoutContainerStore } from "@/stores/useLayoutContainerStore";

const props = defineProps({
  fullHeight: {
    type: Boolean,
    default: true
  },
  padding: {
    type: Boolean,
    default: true
  }
});

const { containerState } = useLayoutContainerStore();
</script>

<template>
  <t-card
    :class="{
      'card-panel': true,
      'global-card-container-shadow': true,
      'h-100': props.fullHeight,
      padding: props.padding
    }" class="tb"
  >
    <div v-if="$slots.title" class="card-panel-title">
      <div>
        <a-typography-title :level="5" style="margin-bottom: 0px">
          <slot name="title"></slot>
        </a-typography-title>
      </div>
      <div>
        <a-typography-text>
          <slot name="operator"></slot>
          <slot v-if="containerState.isDesignMode" name="operator-design"></slot>
        </a-typography-text>
      </div>
    </div>
    <div class="card-panel-content">
      <slot name="body"></slot>
      <slot v-if="containerState.isDesignMode" name="body-design"></slot>
    </div>
  </t-card>
</template>

<style lang="scss" scoped>
@import "@/assets/global.scss";

.tb{
    background-color: #fff;
    border: 0px solid #fff;
    box-shadow: 0px 1px 4px 0px rgba(0,0,0,0.05) !important;
    margin-left: auto;
    margin-right: auto;
    box-sizing: border-box;
    border-radius : 0px !important;
    color: rgba(0,0,0,0.9);
    transition: box-shadow .3 sease-in 0s;
   
}
.tb .t-card{padding: 0px !important;}
</style>
