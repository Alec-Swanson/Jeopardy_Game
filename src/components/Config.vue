<script>
export default {
  name: "Config",
  data() {
    return {
      players: 3,
      categories: 4
    }
  },
  emits: ["numP", "numC"],
  methods: {
    //emits the number of players and categories when confirm is clicked
    confirm() {
      this.$emit("numP", this.players)
      this.$emit("numC", this.categories)
    }
  }
}
</script>

<template>
  <div class="ConfigTile">
    <div class="Players">
      <h2>Number of players:</h2>
      <select id="players" v-model.number="players">
        <option value="2">2 players</option>
        <option value="3">3 players</option>
        <option value="4">4 players</option>
        <option value="5">5 players</option>
        <option value="6">6 players</option>
      </select>
    </div>
    <div class="Categories">
      <h2>Number of categories:</h2>
      <select id="categories" v-model.number="categories">
        <option value="2">2 categories</option>
        <option value="3">3 categories</option>
        <option value="4">4 categories</option>
        <option value="5">5 categories</option>
        <option value="6">6 categories</option>
      </select>
    </div>
    <button @click="confirm">Confirm</button>
  </div>
</template>

<style scoped>
.ConfigTile {
  position: fixed;
  inset: 10px;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: var(--tile-blue);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  margin: 0 auto 24px;
  padding: 18px 24px;
  text-align: center;
  box-shadow: 0 20px 50px var(--panel-shadow);
  gap: 20px;
}

h2 {
  color: var(--gold-soft);
  font-size: 48px;
  text-shadow: 0 3px 0 rgba(0, 0, 0, 0.5);
}

select {
  width: 250px;
  height: 45px;
  font-size: 18px;
  padding: 8px;
}

.ConfigTile button {
  min-width: 90px;
  padding: 12px 24px;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(255, 255, 255, 0.1);
  color: var(--ink);
  cursor: pointer;
  font-family: inherit;
  font-size: 18px;
  font-weight: 600;
  letter-spacing: 0.3px;
  transition: background 0.15s ease, transform 0.1s ease;
}

.ConfigTile button:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-1px);
}

.ConfigTile button:active {
  transform: translateY(0);
}
</style>