<template>
  <div class="small-class-layout">
    <multi-stream-view
      class="stream-container"
      :max-column="maxColumn"
      :max-row="maxRow"
      :exclude-stream-info-list="excludeStreamInfoList"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, Ref, watch, computed, nextTick, onMounted, onUnmounted } from 'vue';
import { storeToRefs } from 'pinia';
import { StreamInfo, useRoomStore } from '../../../stores/room';
import { useBasicStore } from '../../../stores/basic';
import { LAYOUT } from '../../../constants/render';
import eventBus from '../../../hooks/useMitt';
import Whiteboard from '../../Whiteboard/index.vue';
import useStreamContainerHooks from './useStreamContainerHooks';
import SingleStreamView from '../../Stream/SingleStreamView/index.vue';
import MultiStreamView from '../../Stream/MultiStreamView';
import ArrowStroke from '../../common/ArrowStroke.vue';
import { TUIVideoStreamType, TRTCVideoStreamType } from '@tencentcloud/tuiroom-engine-js';
import { roomService, MetricsKey, roomEngine } from '../../../services';

// room store info
const roomStore = useRoomStore();
const {
  remoteScreenStream,
  localScreenStream,
} = storeToRefs(roomStore);

// layout setting
const maxColumn = ref(1);
const maxRow = ref(Infinity);

// stream setting
const excludeStreamInfoList = computed(() => {
  const excludeStreamList: StreamInfo[] = [];
  if (remoteScreenStream.value) {
    excludeStreamList.push(remoteScreenStream.value);
  }
  if (localScreenStream.value) {
    excludeStreamList.push(localScreenStream.value);
  }

  return excludeStreamList;
});

</script>

<style lang="scss" scoped>

.small-class-layout {
  width: 100%;
  height: 100%;
  margin-top: 69px;
  margin-bottom: 80px;
  overflow: hidden;
}

.stream-container {
  background-color: transparent;
  margin-left: 20px;
  width: 214px;
  height: 100%;
}

</style>
