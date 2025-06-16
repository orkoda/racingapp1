<template>
  <div class="race">
    <!-- Left side - Race info -->
    <div class="race-info-container">
      <div class="race-info-block">
        <div class="lap-info">
          <v-icon icon="mdi-flag-checkered" class="lap-icon"></v-icon>
          <span class="lap-text">LAP {{ globalStore.activeRace.currentLap }}/{{ lapText }}</span>
          <span class="last-lap-text" v-if="globalStore.activeRace.currentLap > 1">LAST LAP</span>
        </div>
        
        <div class="time-info">
          <div class="time-row">
            <v-icon icon="mdi-timer-outline" class="time-icon"></v-icon>
            <span class="time-label">TOTAL</span>
            <span class="time-value">{{ msToHMS(globalStore.activeRace.totalTime) }}</span>
          </div>
          
          <div class="time-row">
            <v-icon icon="mdi-timer-sync-outline" class="time-icon"></v-icon>
            <span class="time-label">CURRENT</span>
            <span class="time-value">{{ msToHMS(globalStore.activeRace.time) }}</span>
          </div>
          
          <div class="time-row">
            <v-icon icon="mdi-timer-star-outline" class="time-icon"></v-icon>
            <span class="time-label">BEST</span>
            <span class="time-value">{{ msToHMS(globalStore.activeRace.bestLap) }}</span>
          </div>
          
          <div class="time-row">
            <v-icon icon="mdi-map-marker-check" class="time-icon"></v-icon>
            <span class="time-label">CHECKPOINT</span>
            <span class="time-value">{{ globalStore.activeRace.currentCheckpoint }}/{{ globalStore.activeRace.totalCheckpoints }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Right side - Leaderboard -->
    <div class="leaderboard-container">
      <div 
        class="leaderboard-item"
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
        <div class="racer-name">{{ racer.RacerName }}</div>
        <div class="racer-time" v-if="index === 0 && racer.Finished">
          {{ translate('winner') }}
        </div>
        <div class="racer-time" v-else-if="racer.Finished">
          {{ translate('finished') }}
        </div>
        <div class="racer-time" v-else-if="index !== globalStore.activeRace.position - 1">
          {{ getTimeDifference(racers[globalStore.activeRace.position - 1], racer) }}
        </div>
      </div>
    </div>

    <!-- Ghosted indicator -->
    <div class="ghosted-indicator" v-if="globalStore.activeRace.ghosted">
      <v-icon icon="mdi-ghost-outline"></v-icon>
      <span>GHOSTED</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useGlobalStore } from "@/store/global";
import { msToHMS } from "@/helpers/msToHMS";
import { computed } from "vue";
import { translate } from "@/helpers/translate";
import { ActiveRacer } from "../../store/types";

const globalStore = useGlobalStore();

const props = defineProps<{
  racers?: ActiveRacer[];
}>();

const racers = computed(() => props.racers || globalStore.activeRace.positions || []);

const shortenedRacers = computed(() =>
  racers.value?.slice(
    0,
    globalStore.baseData?.data?.hudSettings?.maxPositions || 5
  )
);

const lapText = computed(() => {
  if (globalStore.activeRace.totalLaps === 0) return "∞";
  else if (globalStore.activeRace.totalLaps === -1) return "∞";
  return globalStore.activeRace.totalLaps.toString();
});

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

  // Positive if racer1 is ahead, negative if racer2 is ahead
  return timeDifference;
};
</script>

<style scoped lang="scss">
.race {
  position: absolute;
  top: 20px;
  left: 20px;
  right: 20px;
  z-index: 100;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  font-family: 'Roboto', sans-serif;
  pointer-events: none;
}

.race-info-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.race-info-block {
  background: rgba(0, 0, 0, 0.8);
  border-radius: 8px;
  padding: 15px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.lap-info {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.lap-icon {
  color: #4CAF50;
  font-size: 18px;
}

.lap-text {
  color: #4CAF50;
  font-weight: 600;
  font-size: 14px;
  text-transform: uppercase;
}

.last-lap-text {
  color: #888;
  font-size: 12px;
  text-transform: uppercase;
  margin-left: auto;
}

.time-info {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.time-row {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 200px;
}

.time-icon {
  color: #4CAF50;
  font-size: 16px;
  width: 20px;
}

.time-label {
  color: #fff;
  font-weight: 500;
  font-size: 12px;
  text-transform: uppercase;
  min-width: 80px;
}

.time-value {
  color: #4CAF50;
  font-weight: 600;
  font-size: 14px;
  font-family: 'Roboto Mono', monospace;
  margin-left: auto;
}

.leaderboard-container {
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 250px;
}

.leaderboard-item {
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

.leaderboard-item.current-player {
  background: rgba(76, 175, 80, 0.3);
  border-color: #4CAF50;
  box-shadow: 0 0 10px rgba(76, 175, 80, 0.3);
}

.leaderboard-item.first-place {
  background: rgba(255, 193, 7, 0.2);
  border-color: #FFC107;
}

.leaderboard-item.second-place {
  background: rgba(158, 158, 158, 0.2);
  border-color: #9E9E9E;
}

.leaderboard-item.third-place {
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

.racer-time {
  color: #4CAF50;
  font-weight: 600;
  font-size: 11px;
  font-family: 'Roboto Mono', monospace;
  min-width: 60px;
  text-align: right;
}

.ghosted-indicator {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(156, 39, 176, 0.9);
  color: #fff;
  padding: 8px 16px;
  border-radius: 20px;
  font-weight: 600;
  font-size: 14px;
  text-transform: uppercase;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

/* Responsive adjustments */
@media (max-width: 1200px) {
  .race {
    left: 10px;
    right: 10px;
  }
  
  .leaderboard-container {
    min-width: 200px;
  }
  
  .time-row {
    min-width: 180px;
  }
}

@media (max-width: 768px) {
  .race {
    flex-direction: column;
    gap: 15px;
  }
  
  .leaderboard-container {
    min-width: auto;
    width: 100%;
  }
}
</style>
</template>