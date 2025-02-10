<!--
  * Name: Switchtheme
  * Usage:
  * Use <switch-theme /> in template
  *
-->
<template>
  <icon-button
    v-if="visible && switchThemeConfig.visible"
    :title="t('Switch Theme')"
    :layout="IconButtonLayout.HORIZONTAL"
    :icon="SwitchThemeIcon"
    @click-icon="handleSwitchTheme"
  />
</template>

<script setup lang="ts">
import { withDefaults, defineProps } from 'vue';
import IconButton from './base/IconButton.vue';
import SwitchThemeIcon from './icons/SwitchThemeIcon.vue';
import { IconButtonLayout } from '../../constants/room';
import { useI18n } from '../../locales';
import { roomService } from '../../services';
import { useBasicStore } from '../../stores/basic';
import { useUIKit } from '@tencentcloud/uikit-base-component-vue3';

const { t } = useI18n();
const basicStore = useBasicStore();
const { theme, setTheme } = useUIKit();

const switchThemeConfig = roomService.getComponentConfig('SwitchTheme');

interface Props {
  visible?: boolean;
}

withDefaults(defineProps<Props>(), {
  visible: true,
});

function handleSwitchTheme() {
  if (theme.value) {
    setTheme(theme.value === 'light' ? 'dark' : 'light');
  } else {
    roomService.setTheme(
      basicStore.defaultTheme === 'light' ? 'dark' : 'light'
    );
  }
}
</script>

<style></style>
