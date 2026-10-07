<script>
export default {
  name: "PlayerCard",
  props: {
    player: {
      type: Object,
      required: true
    }
  },
  emits: ["adjust"]
}
</script>

<template>
  <div class="player">
    <div class="name">{{ player.name }}</div>
    <div class="score">${{ player.score }}</div>
  </div>
</template>

<style scoped>
.player {
  background: var(--tile-blue);
  border-radius: 8px;
  padding: 12px 16px;
  min-width: 150px;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.name {
  font-size: 13px;
  color: var(--ink-dim);
  font-weight: 600;
  background: transparent;
  border: none;
  text-align: center;
  width: 100%;
  font-family: inherit;
}

.score {
  font-family: Georgia, serif;
  font-size: 28px;
  font-weight: 700;
  color: var(--gold-soft);
  margin: 4px 0 8px;
}
</style>
