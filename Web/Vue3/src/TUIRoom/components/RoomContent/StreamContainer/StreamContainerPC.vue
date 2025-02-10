<template>
  <div id="streamContainer" class="stream-container">
    <div class="stream-container-left">
      <single-stream-view
        v-if="enlargeStream && !isMaster"
        class="enlarged-stream-container"
        :stream-info="enlargeStream"
      />
      <Whiteboard v-if="isMaster" class="whiteboard" />
    </div>
    <div :class="['stream-container-right', `${showSideList ? '' : 'hide-list'}`]">
      <large-class-layout v-if="currentClassType === ClassType.LargeClass"></large-class-layout>
      <small-class-layout v-if="currentClassType === ClassType.SmallClass"></small-class-layout>
      <one-to-one-class-layout v-if="currentClassType === ClassType.OneToOneClass"></one-to-one-class-layout>
    </div>
    <arrow-stroke
      v-if="isSideListLayout && (showRoomTool || showSideList)"
      :class="[`arrow-stroke-${arrowDirection}`]"
      :stroke-position="strokePosition"
      :arrow-direction="arrowDirection"
      :has-stroke="showSideList"
      @click-arrow="handleClickIcon"
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
import ArrowStroke from '../../common/ArrowStroke.vue';
import { TRTCVideoStreamType } from '@tencentcloud/tuiroom-engine-js';
import { roomService, MetricsKey, roomEngine } from '../../../services';
import { ClassType } from '../../../utils/common';
import LargeClassLayout from '../ClassLayout/LargeClassLayout.vue';
import SmallClassLayout from '../ClassLayout/SmallClassLayout.vue';

const roomStore = useRoomStore();
const {
  masterCameraStream,
  localStream,
  streamList,
  localScreenStream,
  remoteScreenStream,
  isMaster,
  isSharingWhiteboard,
  currentClassType,
} = storeToRefs(roomStore);
const basicStore = useBasicStore();
const { showRoomTool, layout } = storeToRefs(basicStore);

const { getStreamKey } = useStreamContainerHooks();


const isSideListLayout = computed(
  () =>
    [LAYOUT.RIGHT_SIDE_LIST, LAYOUT.TOP_SIDE_LIST].indexOf(layout.value) >= 0
);

const fixedStream: Ref<StreamInfo | null> = ref(null);
const enlargeStream = computed(() => {
  if (!isSideListLayout.value) {
    return null;
  }
  if (fixedStream.value) {
    return fixedStream.value;
  }
  if (localScreenStream.value) {
    return localScreenStream.value;
  }
  if (remoteScreenStream.value) {
    return remoteScreenStream.value;
  }
  if (teacherStream.value) {
    return teacherStream.value;
  }
  return localStream.value;
});

const teacherStream = computed(() => {
  if (!isSideListLayout.value) {
    return null;
  }

  return masterCameraStream.value;
});

watch(
  () =>
    localScreenStream.value?.hasVideoStream
    || remoteScreenStream.value?.hasVideoStream,
  (val, oldVal) => {
    if (val && !oldVal) {
      fixedStream.value = null;
      if (!isSideListLayout.value) {
        basicStore.setLayout(LAYOUT.RIGHT_SIDE_LIST);
      }
    }
  }
);

watch(
  () => streamList.value.map((stream: StreamInfo) => getStreamKey(stream)),
  (val, oldVal) => {
    if (fixedStream.value) {
      const fixedStreamKey = getStreamKey(fixedStream.value);
      if (!val.includes(fixedStreamKey) && oldVal.includes(fixedStreamKey)) {
        fixedStream.value = null;
      }
    }
  }
);

watch(
  () => isMaster.value,
  async (val) => {
    if (val) {
      roomService.dataReportManager.reportCount(MetricsKey.startSharingWhiteboard);
      await nextTick();
      await startShareWhiteboard();
    }
  },
);

watch(
  () => currentClassType.value,
  (val) => {
    if (val === ClassType.SmallClass) {
      document.documentElement.style.setProperty('--right-region-width', '254px');
    } else {
      document.documentElement.style.setProperty('--right-region-width', '400px');
    }
  },
);

/**
 * ----- The following processing stream layout ---------
 **/
const showSideList = ref(true);
const arrowDirection = computed(() => {
  let arrowDirection = 'right';
  if (layout.value === LAYOUT.TOP_SIDE_LIST) {
    arrowDirection = showSideList.value ? 'up' : 'down';
  }
  if (layout.value === LAYOUT.RIGHT_SIDE_LIST) {
    arrowDirection = showSideList.value ? 'right' : 'left';
  }
  return arrowDirection;
});
const strokePosition = computed(() => {
  let strokePosition = '';
  if (layout.value === LAYOUT.TOP_SIDE_LIST) {
    strokePosition = 'bottom';
  }
  if (layout.value === LAYOUT.RIGHT_SIDE_LIST) {
    strokePosition = 'left';
  }
  return strokePosition;
});
function handleClickIcon() {
  showSideList.value = !showSideList.value;
}

async function onScreenOrWhiteboardCaptureStopped() {
  if (isSharingWhiteboard.value) {
    roomStore.setIsSharingWhiteboard(false);
  } else {
    await nextTick();
    await startShareWhiteboard();
  }
}

function onScreenShareCancelled() {
  startShareWhiteboard();
}

async function startShareWhiteboard() {
  const canvasElt = document.getElementById('whiteboard');
  if (!canvasElt) {
    return;
  }
  const stream = (canvasElt as HTMLCanvasElement).captureStream(15);
  const videoTrack = stream?.getVideoTracks()[0];
  await roomEngine.instance?.getTRTCCloud().startScreenShare(null, TRTCVideoStreamType.TRTCVideoStreamTypeSub, { videoTrack });
  roomStore.setIsSharingWhiteboard(true);
}

onMounted(() => {
  eventBus.on('screenShareCancelled', onScreenShareCancelled);
  eventBus.on('screenShareStopped', onScreenOrWhiteboardCaptureStopped);
});

onUnmounted(() => {
  eventBus.off('screenShareCancelled', onScreenShareCancelled);
  eventBus.off('screenShareStopped', onScreenOrWhiteboardCaptureStopped);
});

document.documentElement.style.setProperty('--right-region-width', '254px');
</script>

<style lang="scss" scoped>
.arrow-stroke-right {
  position: absolute;
  top: 0;
  right: var(--right-region-width);
}

.arrow-stroke-left {
  position: absolute;
  top: 0;
  right: 12px;
}

.stream-container {
  position: relative;
  display: flex;
  flex-wrap: nowrap;
  place-content: center space-between;
  width: 100%;
  height: 100%;
  overflow: hidden;

  .stream-container-left {
    display: flex;
    flex: 1;
    align-items: center;
    justify-content: center;
    width: calc(100% - var(--right-region-width));
    height: calc(100%);
    padding: 25px 20px;

    .enlarged-stream-container {
      display: flex;
      flex: 1;
      align-items: center;
      justify-content: center;
      width: 100%;
      height: 100%;
      padding: 25px 20px;
    }

    .whiteboard {
      display: flex;
      flex: 1;
      align-items: center;
      justify-content: center;
      width: 100%;
      height: 100%;
      padding: 25px 20px;
    }
  }

  .stream-container-right {
    display: flex;
    flex-direction: column;
    position: relative;
    background-color: var(--bg-color-input);
    width: var(--right-region-width);
    height: 100%;
    padding: 0px;

    &.hide-list {
      position: absolute;
      top: 0;
      right: 0;
      transform: translateX(var(--right-region-width));
    }

    &::before {
      position: absolute;
      top: 10px;
      left: 0;
      width: 100%;
      height: 40px;
      content: '';
      opacity: 0.1;
    }
  }
}
</style>
