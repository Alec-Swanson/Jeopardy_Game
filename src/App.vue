<script>
import JeopardyBoard from './components/JeopardyBoard.vue'
import ScoreBoard from './components/ScoreBoard.vue'
import QuestionBoard from './components/QuestionBoard.vue'
import Config from './components/Config.vue'
import WagerPrompt from './components/WagerPrompt.vue'
import CategoriesData from './categories.json'

const Categories = CategoriesData.trivia_categories

//Function that picks random categories without repeats
function pickRandomCategories(categories, number) {
  const pool = categories
  const picks = []

  while (picks.length < number && pool.length) {
    const randomIndex = Math.floor(Math.random() * pool.length)
    picks.push(pool.splice(randomIndex, 1)[0])
  }

  return picks
}

//Function that builds the categories data structure
function buildSelectedCategories(count) {
  return pickRandomCategories(Categories, count).map(({ id, name }) => ({
    id, name, amt: [100, 200, 300, 400, 500].map(value => ({ value, used: false, outcome: null}))
  }))
}

//Function that builds the player data structure
function buildPlayers(count) {
  return Array.from({length: count}, (_, i) => ({ name: `Player ${i + 1}`, score: 0 }))
}

export default {
  name: "App",
  components: { JeopardyBoard, ScoreBoard, QuestionBoard, Config, WagerPrompt },
  data() {
    return {
      playerCount: null,
      categories: [],
      players: [],
      currentPlayer: 0,
      currentCategory: '',
      currentCategoryId: null,
      currentValue: 0,
      currentCategoryIndex: null,
      currentAmtIndex: null,
      currentSelectionId: 0,
      boardLocked: false,
      show: true,
      doubleJeopardyActive: false,
      awaitingWager: false,
      sessionToken: null
    }
  },
  computed: {
    //Checks if all tiles are used
    allTilesUsed() {
      if (!this.categories.length) return false
      return this.categories.every(category => category.amt.every(tile => tile.used))
    },
    //Returns the winner when all tiles have been used
    winnerName() {
      if (!this.allTilesUsed) return null

      const topScore = Math.max(...this.players.map(p => p.score))
      const leaders = this.players.filter(p => p.score === topScore)

      if (leaders.length > 1) {
        return `${leaders.map(p => p.name).join(' & ')} (tie)`
      }
      return leaders[0].name
    }
  },
  methods: {
    handleNumP(count) {
      this.playerCount = count
    },
    handleNumC(count) {
      this.startGame(this.playerCount, count)
    },
    /*
    Starts the game by building both data structures, setting the current player index,
    sets the session token, and transitions from the config to game
     */
    async startGame(playerCount, categoryCount) {
      this.players = buildPlayers(playerCount)
      this.categories = buildSelectedCategories(categoryCount)
      this.currentPlayer = 0
      this.sessionToken = await this.fetchSessionToken()
      this.show = false
    },
    /*
    Fetches the session token. A session token is used to prevet two easy or medium questions
    from the same category from being the same
     */
    async fetchSessionToken() {
      const url = `https://opentdb.com/api_token.php?command=request`
      const response = await fetch(url)
      const data = await response.json()
      return data.token
    },
    //sets the information from the selected tile and checks if double jeopardy
    selectTile({ categoryIndex, amtIndex }) {
      const category = this.categories[categoryIndex]
      const amount = this.categories[categoryIndex].amt[amtIndex]
      this.currentCategory = category.name
      this.currentCategoryId = category.id
      this.currentCategoryIndex = categoryIndex
      this.currentAmtIndex = amtIndex
      this.boardLocked = true

      const faceValue = amount.value
      const balance = this.players[this.currentPlayer].score
      this.doubleJeopardyActive = Math.random() < 0.1

      if (!this.doubleJeopardyActive) {
        this.finalizeSelection(faceValue)
        return
      }

      if (balance < faceValue) {
        this.finalizeSelection(faceValue)
      } else {
        this.awaitingWager = true
      }
    },
    //helper for setting the value of the current question in double jeopardy
    submitWager(amount) {
      this.awaitingWager = false
      this.finalizeSelection(amount)
    },
    //Sets the value of the current question
    finalizeSelection(value) {
      this.currentValue = value
      this.currentSelectionId++
    },
    adjustScore({ index, amount }) {
      this.players[index].score += amount
    },
    //assigns the outcome to a tile, unlocks the board, and adjusts the score
    handleAnswered({ correct, value }) {
      const tile = this.categories[this.currentCategoryIndex].amt[this.currentAmtIndex]
      tile.used = true
      tile.outcome = {player: this.currentPlayer, correct}
      this.boardLocked = false

      if (correct) {
        this.adjustScore({ index: this.currentPlayer, amount: value})
      } else {
        this.adjustScore({ index: this.currentPlayer, amount: -value })
        this.currentPlayer = (this.currentPlayer + 1) % this.players.length
      }
    },
    //Unlocks the board if there is an error in getting the question
    handleError() {
      this.boardLocked = false
    }
  }
}
</script>

<template>
  <div class="container">
    <header class="header">
      <h1>Jeopardy</h1>
    </header>

    <Transition name="fade">
      <div v-if="show" key="config">
        <Config @num-p="handleNumP"
                @num-c="handleNumC"
        />
      </div>
      <div v-else key="game">
        <JeopardyBoard :categories="categories"
                       :locked="boardLocked"
                       @select-tile="selectTile"
        />
        <WagerPrompt v-if="awaitingWager"
                     :player="players[currentPlayer].name"
                     :category="currentCategory"
                     :balance="players[currentPlayer].score"
                     @submit-wager="submitWager"
        />
        <QuestionBoard :player="players[currentPlayer] ? players[currentPlayer].name : ''"
                       :category="currentCategory"
                       :category-Id="currentCategoryId"
                       :value="currentValue"
                       :session-token="sessionToken"
                       :selection-Id="currentSelectionId"
                       :double-jeopardy="doubleJeopardyActive"
                       :winner="winnerName"
                       @answered="handleAnswered"
                       @load-error="handleError"
        />
        <ScoreBoard  :players="players" @adjust-score="adjustScore"/>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
.container {
  max-width: 1100px;
  margin: 0 auto;
  padding: 32px 20px 60px;
}

.header {
  text-align: center;
  margin-bottom: 28px;
}

.header h1 {
  font-family: Georgia, 'Times New Roman', serif;
  font-weight: 700;
  font-style: italic;
  letter-spacing: 0.5px;
  font-size: clamp(32px, 5vw, 52px);
  margin: 0;
  color: var(--gold-soft);
  text-shadow: 0 3px 0 rgba(0, 0, 0, 0.5);
}
</style>
