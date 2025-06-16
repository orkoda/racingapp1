<template>
  <div class="race">
    <!-- Left side - Race info panel exactly like the image -->
    <div class="race-info-panel">
      <!-- Lap section -->
      <div class="lap-section">
        <div class="lap-indicator">
          <v-icon icon="mdi-flag-checkered" class="flag-icon"></v-icon>
          <span class="lap-text">LAP {{ globalStore.activeRace.currentLap }}/{{ lapDisplayText }}</span>
        </div>
        <div class="last-lap-badge" v-if="isLastLap">LAST LAP</div>
      </div>
      
      <!-- Time sections -->
      <div class="time-section">
        <v-icon icon="mdi-timer-outline" class="time-icon"></v-icon>
        <span class="time-label">TOTAL</span>
        <span class="time-value">{{ msToHMS(globalStore.activeRace.totalTime) }}</span>
      </div>
      
      <div class="time-section">
        <v-icon icon="mdi-timer-sync-outline" class="time-icon"></v-icon>
        <span class="time-label">CURRENT</span>
        <span class="time-value">{{ msToHMS(globalStore.activeRace.time) }}</span>
      </div>
      
      <div class="time-section">
        <v-icon icon="mdi-star-outline" class="time-icon"></v-icon>
        <span class="time-label">BEST</span>
        <span class="time-value">{{ msToHMS(globalStore.activeRace.bestLap) }}</span>
      </div>
      
      <div class="time-section">
        <v-icon icon="mdi-map-marker-check-outline" class="time-icon"></v-icon>
        <span class="time-label">CHECKPOINT</span>
        <span class="time-value">{{ globalStore.activeRace.currentCheckpoint }}/{{ globalStore.activeRace.totalCheckpoints }}</span>
      </div>
    </div>

    <!-- Right side - Leaderboard exactly like the image -->
    <div class="leaderboard-panel">
      <div 
        class="leaderboard-row"
        v-for="(racer, index) in shortenedRacers"
        :key="racer.RacerSource"
        :class="{
          'current-player': index === globalStore.activeRace.position - 1
        }"
      >
        <div class="position-badge" :class="getPositionClass(index)">
          {{ index + 1 }}
        </div>
        <div class="racer-info">
          <span class="racer-name">{{ racer.RacerName }}</span>
          <span class="racer-time" v-if="index === 0 && racer.Finished">
            {{ translate('winner') }}
          </span>
          <span class="racer-time" v-else-if="racer.Finished">
            {{ translate('finished') }}
          </span>
          <span class="racer-time" v-else-if="index !== globalStore.activeRace.position - 1">
            {{ getTimeDifference(racers[globalStore.activeRace.position - 1], racer) }}
          </span>
        </div>
      </div>
    </div>

    <!-- Ghosted indicator -->
    <div class="ghosted-indicator" v-if="globalStore.activeRace.ghosted">
      <v-icon icon="mdi-ghost-outline"></v-icon>
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

const lapDisplayText = computed(() => {
  if (globalStore.activeRace.totalLaps === 0) return "∞";
  else if (globalStore.activeRace.totalLaps === -1) return "∞";
  return globalStore.activeRace.totalLaps.toString();
});

const isLastLap = computed(() => {
  return globalStore.activeRace.totalLaps > 0 && 
         globalStore.activeRace.currentLap === globalStore.activeRace.totalLaps;
});

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

/* Left panel - Race info (exactly like the image) */
.race-info-panel {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.lap-section {
  background: rgba(0, 0, 0, 0.85);
  border-radius: 4px;
  padding: 8px 12px;
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 4px;
  border-left: 3px solid #4CAF50;
}

.lap-indicator {
  display: flex;
  align-items: center;
  gap: 6px;
}

.flag-icon {
  color: #4CAF50;
  font-size: 16px;
}

.lap-text {
  color: #4CAF50;
  font-weight: 600;
  font-size: 13px;
  text-transform: uppercase;
}

.last-lap-badge {
  background: rgba(76, 175, 80, 0.2);
  color: #4CAF50;
  padding: 2px 6px;
  border-radius: 3px;
  font-size: 10px;
  font-weight: 600;
  text-transform: uppercase;
  margin-left: auto;
}

.time-section {
  background: rgba(0, 0, 0, 0.85);
  border-radius: 4px;
  padding: 6px 12px;
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 2px;
  min-width: 200px;
}

.time-icon {
  color: #4CAF50;
  font-size: 14px;
  width: 16px;
}

.time-label {
  color: #ffffff;
  font-weight: 500;
  font-size: 11px;
  text-transform: uppercase;
  min-width: 70px;
}

.time-value {
  color: #4CAF50;
  font-weight: 600;
  font-size: 12px;
  font-family: 'Roboto Mono', monospace;
  margin-left: auto;
}

/* Right panel - Leaderboard (exactly like the image) */
.leaderboard-panel {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 220px;
}

.leaderboard-row {
  background: rgba(0, 0, 0, 0.85);
  border-radius: 4px;
  padding: 6px 8px;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: all 0.2s ease;
}

.leaderboard-row.current-player {
  background: rgba(76, 175, 80, 0.3);
  border: 1px solid #4CAF50;
}

.position-badge {
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
}

.position-badge.first-place {
  background: #4CAF50;
}

.position-badge.second-place {
  background: #4CAF50;
}

.position-badge.third-place {
  background: #4CAF50;
}

.position-badge.other-place {
  background: #4CAF50;
}

.racer-info {
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

.racer-time {
  color: #ffffff;
  font-weight: 600;
  font-size: 10px;
  font-family: 'Roboto Mono', monospace;
  text-align: right;
  min-width: 60px;
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
}

/* Responsive adjustments */
@media (max-width: 1200px) {
  .race {
    left: 10px;
    right: 10px;
  }
  
  .leaderboard-panel {
    min-width: 180px;
  }
  
  .time-section {
    min-width: 180px;
  }
}

@media (max-width: 768px) {
  .race {
    flex-direction: column;
    gap: 10px;
  }
  
  .leaderboard-panel {
    min-width: auto;
    width: 100%;
  }
}
</style>
</template>