<template>
  <div class="racers-holder">
    <div
      class="racer-row"
      v-for="(racer, index) in shortenedRacers"
      :key="racer.RacerSource"
      :class="{
        'current-player': index === globalStore.activeRace.position - 1
      }"
    >
      <div class="position-number" :class="getPositionClass(index)">
        {{ index + 1 }}
      </div>
      <div class="racer-details">
        <span class="racer-name">{{ racer.RacerName }}</span>
        <span class="time-diff" v-if="index === 0 && racer.Finished">{{ translate('winner') }}</span>
        <span class="time-diff" v-else-if="racer.Finished">{{ translate('finished') }}</span>
        <span class="time-diff" v-else-if="index !== globalStore.activeRace.position - 1">
          {{ getTimeDifference(racers[globalStore.activeRace.position - 1], racer) }}
        </span>
      </div>
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

const getPositionClass = (index: number) => {
  if (index === 0) return 'first-place';
  if (index === 1) return 'second-place';
  if (index === 2) return 'third-place';
  return 'other-place';
};

const formatTimeDifference = (timeDiffMs: number): string => {
  if (timeDiffMs === 0) {
    return "0.000";
  }

  const isAhead = timeDiffMs > 0;
  const absoluteDiffSeconds = Math.abs(timeDiffMs) / 1000;
  const formattedTime = absoluteDiffSeconds.toFixed(3);

  return isAhead ? `+${formattedTime}` : `-${formattedTime}`;
};

const getTimeDifference = (racer1: ActiveRacer, racer2: ActiveRacer) => {
  const racer1Checkpoints = racer1.CheckpointTimes.length;
  const racer2Checkpoints = racer2.CheckpointTimes.length;

  if (racer1Checkpoints === 0 || racer2Checkpoints === 0) {
    return '';
  }

  const lastCommonCheckpoint = Math.min(racer1Checkpoints, racer2Checkpoints) - 1;
  const racer1Time = racer1.CheckpointTimes[lastCommonCheckpoint].time;
  const racer2Time = racer2.CheckpointTimes[lastCommonCheckpoint].time;

  return formatTimeDifference(racer2Time - racer1Time);
};
</script>

<style scoped lang="scss">
.racers-holder {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 220px;
}

.racer-row {
  background: rgba(0, 0, 0, 0.85);
  border-radius: 4px;
  padding: 6px 8px;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: all 0.2s ease;
}

.racer-row.current-player {
  background: rgba(76, 175, 80, 0.3);
  border: 1px solid #4CAF50;
}

.position-number {
  width: 20px;
  height: 20px;
  border-radius: 2px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 11px;
  color: #000;
  flex-shrink: 0;
  background: #4CAF50;
}

.racer-details {
  display: flex;
  flex: 1;
  justify-content: space-between;
  align-items: center;
  gap: 8px;
}

.racer-name {
  color: #ffffff;
  font-weight: 500;
  font-size: 12px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex: 1;
}

.time-diff {
  color: #ffffff;
  font-weight: 600;
  font-size: 10px;
  font-family: 'Roboto Mono', monospace;
  text-align: right;
  min-width: 60px;
}
</style>
</template>