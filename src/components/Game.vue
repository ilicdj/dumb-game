<script>
export default {
  name: 'Game',
  data() {
    return {
      count: Math.round(Math.random() * 4 + 1),
      gameStarted: false,
      gameOver: false,
      countdown: 3,
      currentClickCount: 0,
    }
  },
  methods: {
    startCountDown() {
      this.gameStarted = true
      const countdownInterval = setInterval(() => {
        if (this.countdown > 0) {
          this.countdown--
        } else {
          clearInterval(countdownInterval)
          this.startGame()
        }
      }, 1000)
    },
    startGame() {
      const interval = setInterval(() => {
        if (this.count > 0 && this.count < 15) {
          if (this.count % 2) {
            this.count--
          } else {
            this.count -= 2
          }
        } else if (this.count == 0) {
          this.gameOver = true
          clearInterval(interval)
        } else if (this.count == 15) {
          this.gameOver = true
          clearInterval(interval)
        }
      }, 400)
    },
    incrementCount() {
      if (this.count !== 0 && this.countdown == 0 && this.count !== 15) {
        this.count++
        this.currentClickCount++
        this.$emit('update-click-count', this.currentClickCount)
      }
    },
    exitGame() {
      this.countdown = 3
      this.gameStarted = false
      this.gameOver = false
      this.count = Math.round(Math.random() * 4 + 1)
      // Reset click count when exiting
      this.$emit('update-click-count', 0)
    },
  },
  computed: {
    gameMessage() {
      switch (this.count) {
        case 0:
          return 'You Lost'
          break
        case 15:
          return 'You Won'
          break

        default:
          return 'Lose if it reaches 0, win if you reach 15'
          break
      }
    },
  },
}
</script>

<template>
  <div id="game" :class="{ started: gameStarted }" @click="incrementCount">
    <button id="startGameBtn" @click="startCountDown" v-show="!gameStarted">
      Start Game
    </button>
    <p id="rule" :class="{ started: gameStarted }">{{ gameMessage }}</p>
    <p id="count" v-show="gameStarted" :class="{ started: gameStarted }">
      {{ count }}
    </p>
    <p id="countdown" v-show="gameStarted && countdown > 0">{{ countdown }}</p>
    <div class="btn-container">
      <button v-show="gameOver" @click="exitGame">Restart Game</button>
    </div>
  </div>
</template>

<style scoped lang="scss">
@import '../scss/variables.scss';
/* Game */
#game {
  user-select: none;
  position: relative;
  height: 100vh;
  width: 100%;
  background-color: transparent;
  &.started {
    position: fixed;
    top: 0;
    left: 0;
  }
  #startGameBtn {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: transparent;
    outline: 0;
    border: 2px solid #f4f4f3;
    font-weight: bold;
    font-size: clamp(1.5rem, 1vw, 4rem);
    color: #f4f4f3;
    padding: 20px 50px;
    cursor: pointer;
    transition: 0.2s all ease;
    font-family: $font-text;
  }
  #rule {
    font-family: $font-text;
    text-align: center;
    padding-top: 50px;
    font-size: clamp(1.5rem, 1vw, 4rem);
    color: $white;
    &.started {
      position: relative;
      z-index: -1;
    }
  }
  #count {
    font-family: $font-text;
    display: none;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 6rem;
    &.started {
      display: block;
    }
  }
  #countdown {
    font-family: $font-text;
    position: absolute;
    top: 25%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 6rem;
    color: $white;
  }
  .btn-container {
    position: absolute;
    bottom: 5%;
    display: flex;
    justify-content: center;
    width: 100%;
    gap: 30px;
    button {
      background-color: transparent;
      outline: 0;
      border: 2px solid #f4f4f3;
      font-weight: bold;
      font-size: clamp(1.5rem, 1vw, 4rem);
      color: #f4f4f3;
      padding: 20px 50px;
      cursor: pointer;
      transition: 0.2s all ease;
      font-family: $font-text;
    }
  }
}
</style>
