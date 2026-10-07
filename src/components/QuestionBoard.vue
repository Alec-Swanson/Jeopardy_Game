<script>
export default {
  name: "QuestionBoard",
  props: {
    player: String,
    category: String,
    categoryId: Number,
    value: Number,
    sessionToken: {
      type: String,
      default: null,
    },
    selectionId: {
      type: Number,
      default: 0
    },
    doubleJeopardy: {
      type: Boolean,
      default: false
    },
    winner: {
      type: String,
      default: null
    }
  },
  emits: ['answered', 'load-error'],
  data() {
    return {
      questionText: '',
      correctAnswer: '',
      answered: false,
      wasCorrect: null,
      loading: false,
      error: null,
      lastFetchTime: 0
    }
  },
  computed: {
    notSelected(){
      return this.value === 0 || (this.answered && !this.wasCorrect)
    }
  },
  watch: {
    selectionId(newId) {
      if (newId) this.fetchQuestion()
    }
  },
  methods: {
    //calculates the difficulty string based on the value of the selected tile
    getDifficulty(value) {
      const difficulties = [ 'easy', 'medium', 'hard' ]

      switch (value) {
        case 100:
        case 200:
          return difficulties[0]
        case 300:
        case 400:
          return difficulties[1]
        case 500:
          return difficulties[2]
        default:
          return difficulties[0]
      }
    },
    //Fetches a question when a player clicks a tile
    async fetchQuestion() {
      this.loading = true
      this.error = null
      this.questionText = ''
      this.correctAnswer = ''
      this.answered = false
      this.wasCorrect = null

      const difficulty = this.getDifficulty(this.value)
      /*
      Builds the url and adds the session token if it exists. This allows the game to be played if the fetch for
      the session token fails, but with the loss of the guard against repeat questions
       */
      let url = `https://opentdb.com/api.php?amount=1&category=${this.categoryId}&difficulty=${difficulty}&type=boolean`
      if (this.sessionToken !== null) {
        url += `&token=${this.sessionToken}`
      }
      const maxAttempts = 3

      //Allows for 3 attempts at getting a question, waiting for the rate limit in between request
      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        await this.waitForRateLimit()

        try {
          this.lastFetchTime = Date.now()
          const response = await fetch(url)

          if (response.status === 429) continue
          if (!response.ok) throw new Error('Network Error')

          const data = await response.json()
          //Reset the session token if all questions for a difficulty in a category is exhausted
          if (data.response_code === 4 && this.sessionToken) {
            await fetch(`https://opentdb.com/api_token.php?command=reset&token=${this.sessionToken}`)
            continue
          }
          if (data.response_code !== 0 || !data.results.length) {
            throw new Error('No questions for this category')
          }

          //Extract question and answer from response
          this.questionText = this.decodeHtml(data.results[0].question)
          this.correctAnswer = data.results[0].correct_answer
          this.loading = false
          return
        } catch (err) {
          if (attempt >= maxAttempts) {
            this.error = 'Could not load question'
            this.loading = false
            this.$emit('load-error')
            return
          }
        }
      }
    },
    //Waits for the 5 second rate limit of the API
    waitForRateLimit() {
      const wait = 5000
      const elapsed = Date.now() - this.lastFetchTime
      if (this.lastFetchTime && elapsed < wait) {
        return new Promise(resolve => setTimeout(resolve, wait - elapsed))
      }
      return Promise.resolve()
    },
    //Checks if the answer is correct
    checkAnswer(selected) {
      if (this.answered || this.loading || this.error) return

      this.answered = true
      this.wasCorrect = selected === this.correctAnswer
      this.$emit('answered', { correct: this.wasCorrect, value: this.value })
    },
    decodeHtml(data) {
      const el = document.createElement('textarea')
      el.innerHTML = data
      return el.value
    }
  }
}
</script>

<template>
  <div v-if="!winner" class="QuestionBoard">
    <p v-if="doubleJeopardy && !notSelected && !answered" class="djBanner">Double Jeopardy</p>
    <header v-if="notSelected" class="header">{{ player }} to select</header>
    <header v-else class="header">{{ player }} selects {{ category }} for ${{ value }}</header>
    <div v-if="!notSelected || selectionId > 0" class="question">
      <p v-if="loading">Loading question</p>
      <p v-else-if="error">{{ error }}</p>
      <p v-else>{{ questionText }}</p>
    </div>
    <div v-if="!notSelected && !answered" class="answers">
      <button :disabled="loading || error" @click="checkAnswer('True')">True</button>
      <button :disabled="loading || error" @click="checkAnswer('False')">False</button>
    </div>
    <p v-if="answered" :class="wasCorrect ? 'correct' : 'incorrect'">
      {{ wasCorrect ? 'Correct! Pick again' : 'Incorrect!' }}
    </p>
  </div>
  <div v-else class="WinnerBoard">
    <h1>{{ winner }} has won the game</h1>
  </div>
</template>

<style scoped>
.QuestionBoard {
  background: var(--tile-blue);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  max-width: 600px;
  margin: 0 auto 24px;
  padding: 18px 24px;
  text-align: center;
  box-shadow: 0 20px 50px var(--panel-shadow);
}

.WinnerBoard {
  background: var(--tile-blue);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 10px;
  max-width: 600px;
  margin: 0 auto 24px;
  padding: 18px 24px;
  text-align: center;
  box-shadow: 0 20px 50px var(--panel-shadow);
}

h1 {
  font-family: Georgia, 'Times New Roman', serif;
  font-weight: 700;
  font-size: clamp(25px, 2.2vw, 30px);
  color: var(--gold-soft);
  letter-spacing: 0.3px;
  margin: 0;
}

.header {
  font-family: Georgia, 'Times New Roman', serif;
  font-weight: 700;
  font-size: clamp(15px, 2.2vw, 20px);
  color: var(--gold-soft);
  letter-spacing: 0.3px;
  margin: 0;
}

.djBanner {
  color: var(--gold);
  font-weight: 700;
  letter-spacing: 0.6px;
  text-transform: uppercase;
  font-size: 13px;
  margin: 0 0 6px;
}

.answers {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin-top: 16px;
}

.answers button {
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

.answers button:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-1px);
}

.answers button:active {
  transform: translateY(0);
}

@media (max-width: 640px) {
  .QuestionBoard {
    padding: 14px 16px;
  }
}
</style>