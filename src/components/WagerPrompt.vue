<script>
export default {
  name: "WagerPrompt",
  props: {
    player: String,
    category: String,
    balance: Number
  },
  emits: ['submit-wager'],
  data() {
    return {
      wagerInput: null,
      errorMsg: ''
    }
  },
  methods: {
    //Checks the wager amount is valid
    submit() {
      const wager = Number(this.wagerInput)

      if (!wager || wager <= 0) {
        this.errorMsg = "Enter a wager greater that $0"
        return
      }
      if (wager > this.balance) {
        this.errorMsg = `You can't wager more than your balance: $${this.balance}`
        return
      }

      this.errorMsg = ''
      this.$emit('submit-wager', wager)
    }
  }
}
</script>

<template>
  <div class="WagerPrompt">
    <h2>Double Jeopardy!</h2>
    <label for="wager">Double Jeopardy wager (up to ${{ balance }}):</label>
    <input id="wager" type="number" v-model.number="wagerInput" min="1" :max="balance" />
    <p v-if="errorMsg" class="error">{{ errorMsg }}</p>
    <button @click="submit">Confirm wager</button>
  </div>
</template>

<style scoped>
.WagerPrompt {
  background: var(--tile-blue);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  max-width: 600px;
  margin: 0 auto 24px;
  padding: 18px 24px;
  text-align: center;
  box-shadow: 0 20px 50px var(--panel-shadow);
}

h2 {
  color: var(--gold-soft);
  font-family: Georgia, 'Times New Roman', serif;
  margin: 0 0 8px;
}

label {
  display: block;
  margin: 16px 0 8px;
  color: var(--ink-dim);
  font-size: 13px;
}

input {
  width: 140px;
  padding: 8px 12px;
  border-radius: 6px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(0, 0, 0, 0.25);
  color: var(--ink);
  font-size: 16px;
  text-align: center;
}

.error {
  color: #e08a8a;
  font-size: 13px;
  margin-top: 8px;
}

.WagerPrompt button {
  display: block;
  margin: 16px auto 0;
  min-width: 90px;
  padding: 10px 20px;
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(255, 255, 255, 0.1);
  color: var(--ink);
  cursor: pointer;
  font-family: inherit;
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 0.3px;
  transition: background 0.15s ease, transform 0.1s ease;
}

.WagerPrompt button:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-1px);
}
</style>