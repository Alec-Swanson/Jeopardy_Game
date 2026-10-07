<script>
export default {
  name: "Tile",
  props: {
    value: {
      type: Number,
      required: true
    },
    used: {
      type: Boolean,
      default: false
    },
    outcome: {
      type: Object,
      default: null
    },
    locked: {
      type: Boolean,
      default: false
    }
  },
  emits: ["select"],
  computed: {
    //Displays the value or P# on the tile
    displayText() {
      if (this.outcome) {
        return `P${this.outcome.player + 1}`
      }
      return this.used ? "" : "$"+this.value
    }
  }
}
</script>

<template>
  <div class=tile :class="{used, locked, won: outcome && outcome.correct, lost: outcome && !outcome.correct}"
       @click="!used && !locked && $emit('select')">
    <span v-if='displayText'>{{ displayText }}</span>
  </div>
</template>

<style scoped>
.tile {
  background: var(--tile-blue);
  border-radius: 4px;
  min-height: 76px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: Georgia, serif;
  font-weight: 700;
  font-size: clamp(20px, 3vw, 30px);
  color: var(--gold);
  cursor: pointer;
  transition: background 0.15s ease, transform 0.1s ease;
  user-select: none;
}

.tile:hover:not(.used):not(.locked) {
  background: var(--tile-blue-hover);
  transform: scale(1.02);
}

.tile.won {
  background: rgba(127, 214, 154, 0.14);
  color: #7fd99a;
  cursor: default;
}

.tile.lost {
  background: rgba(224, 138, 138, 0.14);
  color: #e08a8a;
  cursor: default;
}

@media (max-width: 640px) {
  .tile {
    min-height: 58px;
    padding: 8px 4px;
  }
}
</style>