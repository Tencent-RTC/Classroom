<template>
  <div class="class-type-panel">
    <div :class="getClassStyle(ClassType.OneToOneClass)" @click="handleOneToOneClass">
      <input v-model="selectedClass" class="item-radio" type="radio" :value="ClassType.OneToOneClass" />
      <div class="item-title"> {{ t('1V1 class') }}
        <div class="item-content">{{ t('1V1 call, specific layout, efficient interaction.')}} </div>
        <div class="item-content">{{ t('Recommended usage scenarios: language teaching, online recruitment, remote business development')}} </div>
      </div>
    </div>
    <div :class="getClassStyle(ClassType.SmallClass)" @click="handleSmallClass">
      <input v-model="selectedClass" class="item-radio" type="radio" :value="ClassType.SmallClass" />
      <div class="item-title"> {{ t('Small class') }}
        <div class="item-content">{{ t('Strong interaction, low latency, support for multiple people on stage at the same time.') }}</div>
        <div class="item-content">{{ t('Recommended usage scenarios: small class teaching, conference discussion, online consultation') }} </div>
      </div>
    </div>
    <div :class="getClassStyle(ClassType.LargeClass)" @click="handleLargeClass">
      <input v-model="selectedClass" class="item-radio" type="radio" :value="ClassType.LargeClass" />
      <div class="item-title"> {{ t('Large class') }}
        <div class="item-content"> {{ t('Many people interacted on stage, and thousands of people watched online.') }} </div>
        <div class="item-content"> {{ t('Recommended usage scenarios: open courses, online training, financial marketing') }} </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { useI18n } from '../../locales';
import { ClassType } from '../../utils/common';

const { t } = useI18n();
const selectedClass = ref(ClassType.OneToOneClass);
const emit = defineEmits(['click']);

const props = defineProps<{
  modelValue: ClassType;
}>();

onMounted(() => {
  selectedClass.value = props.modelValue;
});

function handleOneToOneClass() {
  selectedClass.value = ClassType.OneToOneClass;
  emit('click', selectedClass.value);
};

function handleSmallClass() {
  selectedClass.value = ClassType.SmallClass;
  emit('click', selectedClass.value);
}

function handleLargeClass() {
  selectedClass.value = ClassType.LargeClass;
  emit('click', selectedClass.value);
}

function getClassStyle(classType: ClassType) {
  if (classType === ClassType.OneToOneClass) {
    return 'class-item-hidden';
  }
  return selectedClass.value === classType ? 'class-item-selected' : 'class-item';
}

</script>

<style scoped lang="scss">

.class-type-panel {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: auto;
}

.class-item {
  display: flex;
  height: auto;
  margin-top: 5px;
  background-color: var(--bg-color-input);
  border:2px solid var(--bg-color-input);
  border-radius: 8px;
}

.class-item-selected {
  display: flex;
  height: auto;
  margin-top: 5px;
  background-color: var(--bg-color-input);
  border:2px solid var(--text-color-link);
  border-radius: 8px;
}

.class-item-hidden {
  display: none;
  height: auto;
  margin-top: 5px;
  background-color: var(--bg-color-input);
  border:2px solid var(--bg-color-input);
  border-radius: 8px;
}

.item-radio {
  position: relative;
  top: 5px;
  left: 5px;
  width: 15px;
  height: 15px;
}

.item-title {
  flex: 1;
  margin-top: 5px;
  margin-left: 13px;
  font-family: "PingFang SC", sans-serif;
  font-weight: 400;
  font-size: 14px;
  color: var(-text-color-anti-primary);
}

.item-content {
  font-family: "PingFang SC", sans-serif;
  font-weight: 400;
  font-size: 12px;
  color: var(--text-color-secondary);
}

</style>
