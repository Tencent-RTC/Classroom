<template>
  <div class="large-class-layout">
    <single-stream-view
      class="teacher-stream-container"
      :stream-info="masterCameraStream"
    ></single-stream-view>
    <multi-stream-view
      class="students-stream-container"
      :max-column="maxMultiStreamColumn"
      :max-row="maxMultiStreamRow"
      :exclude-stream-info-list="excludeStreamInfoList"
    ></multi-stream-view>
    <div class="line"></div>
    <div class="chat-style">
      <chat></chat>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, computed } from 'vue';
import { storeToRefs } from 'pinia';
import { StreamInfo, useRoomStore } from '../../../stores/room';
import SingleStreamView from '../../Stream/SingleStreamView/index.vue';
import MultiStreamView from '../../Stream/MultiStreamView';
import { TUIVideoStreamType } from '@tencentcloud/tuiroom-engine-js';
import Chat from '../../Chat/index';

// room store info
const roomStore = useRoomStore();
const {
  masterCameraStream,
  remoteScreenStream,
  localScreenStream,
  cameraStreamList,
} = storeToRefs(roomStore);

// layout setting
const maxMultiStreamColumn = ref(2);
const maxMultiStreamRow = ref(Infinity);
const maxStudentRow = ref(2);
const studentStreamHeight = 100;
const studentStreamMargin = 10;

// stream setting
const excludeStreamInfoList = computed(() => {
  const excludeStreamList: StreamInfo[] = [];
  if (remoteScreenStream.value) {
    excludeStreamList.push(remoteScreenStream.value);
  }
  if (localScreenStream.value) {
    excludeStreamList.push(localScreenStream.value);
  }
  if (masterCameraStream.value) {
    excludeStreamList.push(masterCameraStream.value);
  }
  return excludeStreamList;
});

// data watch
watch(
  () => cameraStreamList.value.length,
  (val) => {
    const masterStream = cameraStreamList.value.filter(stream => stream.streamType === TUIVideoStreamType.kCameraStream && stream.userId === roomStore.masterUserId);
    let row = Math.ceil((val - masterStream.length) / 2);
    if (row > maxStudentRow.value) {
      row = maxStudentRow.value;
    }
    const studentsRegionHeight = row * studentStreamHeight + studentStreamMargin;
    document.documentElement.style.setProperty('--students-region-height', `${studentsRegionHeight}px`);
  },
);

</script>

<style lang="scss" scoped>
.large-class-layout {
  width: 100%;
  height: 100%;
  margin-top: 69px;
  margin-bottom: 80px;
  overflow: hidden;
}

.teacher-stream-container {
  margin-left: 20px;
  margin-right: 20px;
  width: 360px;
  height: 200px;
  border: 2px solid var(--bg-color-input);
}

.students-stream-container {
  margin-left: 20px;
  margin-right: 20px;
  width: 360px;
  height: var(--students-region-height);
  border: 2px solid var(--bg-color-input);
}

.line {
  width: 400px;
  height: 2px;;
  background-color: var(--stroke-color-primary);
}

.chat-style {
  margin-left: 20px;
  margin-right: 20px;
  display: flex;
  flex: 1;
  height: calc(100% - 200px - var(--students-region-height));
  border: 2px solid var(--bg-color-input);
}
</style>
