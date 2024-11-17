<!-- eslint-disable vue/valid-template-root -->
<template>
  <RoomMessageCard :message="props.message" />
</template>
<script setup lang="ts">
import { IMessageModel } from '../TUIRoom';
import { RoomMessageCard } from '../TUIRoom';

interface IProps {
  message: IMessageModel;
}
// eslint-disable-next-line @typescript-eslint/no-unused-vars
const props = withDefaults(defineProps<IProps>(), {
  message: () => ({} as IMessageModel),
});
</script>
<style lang="scss" scoped>
/* stylelint-disable-next-line no-empty-source */
</style>
