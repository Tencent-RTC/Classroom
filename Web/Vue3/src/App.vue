<script setup lang="ts">
import { UIKitProvider } from '@tencentcloud/uikit-base-component-vue3';
</script>

<template>
  <UIKitProvider theme="light">
    <router-view></router-view>
  </UIKitProvider>
</template>

<style lang="scss">
  html,body,#app{
        height: 100%;
    }
  #app {
    overflow: hidden;
  }
</style>
