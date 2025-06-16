<template>
  <div class="racers-holder">
    <div
      class="racer-item"
      v-for="(racer, index) in shortenedRacers"
      :key="racer.RacerSource"
      :class="{
        'current-player': index === globalStore.activeRace.position - 1,
        'first-place': index === 0,
        'second-place': index === 1,
        'third-place': index === 2
      }"
    >
      <div class="position-number">{{ index + 1 }}</div>
      <span class="racer-name">{{ racer.RacerName }}</span>
      <span class="time-difference" v-if="index === 0 && racer.Finished">{{ translate('winner') }}</span>
      <span class="time-difference" v-else-if="racer.Finished">{{ translate('finished') }}</span>
      <span class="time-difference" v-else-if="index !== globalStore.activeRace.position - 1">
        {{ getTimeDifference(racers[globalStore.activeRace.position - 1], racer) }}
      </span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useGlobalStore } from "@/store/global";
import { ActiveRacer } from "../../store/types";
import { computed } from "vue";
import { translate } from "@/helpers/translate";

const props = defineProps<{
  racers: ActiveRacer[];
}>();

const globalStore = useGlobalStore();
const shortenedRacers = computed(() =>
  props.racers?.slice(
    0,
    globalStore.baseData?.data?.hudSettings?.maxPositions || 5
  )
);

const formatTimeDifference = (timeDiffMs: number): string => {
  if (timeDiffMs === 0) {
    return "0.000";
  }

  const isAhead = timeDiffMs > 0;
  const absoluteDiffSeconds = Math.abs(timeDiffMs) / 1000;

  // Format to 3 decimal places
  const formattedTime = absoluteDiffSeconds.toFixed(3);

  return isAhead ? `+${formattedTime}` : `-${formattedTime}`;
};

const getTimeDifference = (racer1: ActiveRacer, racer2: ActiveRacer) => {
  const racer1Checkpoints = racer1.CheckpointTimes.length;
  const racer2Checkpoints = racer2.CheckpointTimes.length;

  if (racer1Checkpoints === 0 || racer2Checkpoints === 0) {
    return ''; // Not enough data to compare
  }

  const lastCommonCheckpoint = Math.min(racer1Checkpoints, racer2Checkpoints) - 1;
  const racer1Time = racer1.CheckpointTimes[lastCommonCheckpoint].time;
  const racer2Time = racer2.CheckpointTimes[lastCommonCheckpoint].time;

  const timeDifference = formatTimeDifference(racer2Time - racer1Time);

  return timeDifference;
};
</script>

<style scoped lang="scss">
.racers-holder {
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 250px;
}

.racer-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 12px;
  background: rgba(0, 0, 0, 0.7);
  border-radius: 6px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  transition: all 0.3s ease;
}

.racer-item.current-player {
  background: rgba(76, 175, 80, 0.3);
  border-color: #4CAF50;
  box-shadow: 0 0 10px rgba(76, 175, 80, 0.3);
}

.racer-item.first-place {
  background: rgba(255, 193, 7, 0.2);
  border-color: #FFC107;
}

.racer-item.second-place {
  background: rgba(158, 158, 158, 0.2);
  border-color: #9E9E9E;
}

.racer-item.third-place {
  background: rgba(205, 127, 50, 0.2);
  border-color: #CD7F32;
}

.position-number {
  background: #4CAF50;
  color: #000;
  font-weight: 700;
  font-size: 12px;
  width: 24px;
  height: 24px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.first-place .position-number {
  background: #FFC107;
}

.second-place .position-number {
  background: #9E9E9E;
}

.third-place .position-number {
  background: #CD7F32;
}

.racer-name {
  color: #fff;
  font-weight: 500;
  font-size: 13px;
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.time-difference {
  color: #4CAF50;
  font-weight: 600;
  font-size: 11px;
  font-family: 'Roboto Mono', monospace;
  min-width: 60px;
  text-align: right;
}
</style>
</template>