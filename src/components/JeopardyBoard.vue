<script>
import Tile from "./Tile.vue";
export default {
  name: "JeopardyBoard",
  components: {Tile},
  props: {
    categories: {
      type: Array,
      required: true
    },
    locked: {
      type: Boolean,
      default: false
    }
  },
  emits: ['select-tile'],
  computed: {
    rowCount() {
      return this.categories.length ? this.categories[0].amt.length : 0;
    }
  },
  methods: {
    //emits the tile indices when a tile is clicked
    handleSelect(cIndex, tIndex) {
      this.$emit('select-tile', {categoryIndex: cIndex, amtIndex: tIndex});
    }
  }
}
</script>

<template>
  <div class="board" :style="{'--cols':categories.length}">
    <div v-for="(category,ci) in categories" :key="'cat-'+ ci" class="cat-cell">{{ category.name }}</div>
    <template v-for="rowIndex in rowCount" :key="'row-'+ rowIndex">
      <Tile
          v-for="(category,ci) in categories"
          :key="'tile-'+ci+'-'+rowIndex"
          :value="category.amt[rowIndex - 1].value"
          :used="category.amt[rowIndex - 1].used"
          :outcome="category.amt[rowIndex - 1].outcome"
          :locked="locked"
          @select="handleSelect(ci, rowIndex - 1)"/>
    </template>
  </div>
</template>

<style scoped>
.board {
  display: grid;
  grid-template-columns: repeat(var(--cols, 4), 1fr);
  gap: 6px;
  background: var(--board-navy-deep);
  padding: 6px;
  border-radius: 10px;
  box-shadow: 0 20px 50px var(--panel-shadow);
}

.cat-cell {
  background: var(--tile-blue);
  color: var(--gold-soft);
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 14px 8px;
  min-height: 76px;
  font-weight: 700;
  font-size: clamp(12px, 1.6vw, 16px);
  line-height: 1.25;
  border-radius: 4px;
  letter-spacing: 0.2px;
}

@media (max-width: 640px) {
  .cat-cell {
    min-height: 58px;
    padding: 8px 4px;
  }
}
</style>