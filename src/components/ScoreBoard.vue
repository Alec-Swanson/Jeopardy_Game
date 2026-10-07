<script>
import PlayerCard from './PlayerCard.vue'

export default {
  name: "ScoreBoard",
  components: { PlayerCard },
  props: {
    players: {
      type: Array,
      required: true
    }
  },
  emits: ["adjust-score"]
}
</script>

<template>
  <div class="scoreboard">
    <PlayerCard
        v-for="(player, index) in players"
        :key="'player-' + index"
        :player="player"
        @adjust="amount => $emit('adjust-score', { index, amount })"
    />
  </div>
</template>

<style scoped>
.scoreboard {
  display: flex;
  justify-content: center;
  gap: 14px;
  margin: 26px 0 4px;
  flex-wrap: wrap;
}
</style>
