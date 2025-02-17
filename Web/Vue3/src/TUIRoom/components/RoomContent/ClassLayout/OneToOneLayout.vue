<template>
  <div class="one-to-one-class-layout">
    <single-stream-view
      class="stream-container"
      :stream-info="masterCameraStream"
    ></single-stream-view>
    <single-stream-view
      class="stream-container"
      :stream-info="masterCameraStream"
    ></single-stream-view>
    <div class="line"></div>
    <div class="chat-style">
      <chat></chat>
    </div>
  </div>
</template>

<script setup lang="ts">
import { storeToRefs } from 'pinia';
import { useRoomStore } from '../../../stores/room';
import SingleStreamView from '../../Stream/SingleStreamView/index.vue';
import Chat from '../../Chat/index';

// room store info
const roomStore = useRoomStore();
const {
  masterCameraStream,
} = storeToRefs(roomStore);

</script>

<style lang="scss" scoped>
.one-to-one-class-layout {
  width: 100%;
  height: 100%;
  margin-top: 69px;
  margin-bottom: 80px;
  overflow: hidden;
}

.stream-container {
  margin-left: 20px;
  width: var(--teacher-stream-width);
  height: var(--teacher-stream-height);
  border: 2px solid var(--bg-color-input);
}

.line {
  width: 400px;
  height: 2px;;
  background-color: var(--stroke-color-primary);
}

.chat {
  display: flex;
  flex: 1;
  margin-left: 20px;
  height: calc(100% - 200px - var(--students-region-height));
  border: 2px solid var(--bg-color-input);
}
</style>
