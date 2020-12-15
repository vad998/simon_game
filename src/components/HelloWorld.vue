<template>
  <div class="home">
    <div class="squares" :class="{error: error, victory}">
      <div 
        class="square square_blue"
        @click="squareClick(0)"
        :class="{selection: selectionSquare[0]}"
      ></div>
      <div 
        class="square square_red"
        @click="squareClick(1)"
        :class="{selection: selectionSquare[1]}"
      ></div>
      <div 
        class="square square_yellow"
        @click="squareClick(2)"
        :class="{selection: selectionSquare[2]}"
      ></div>
      <div 
        class="square square_green"
        @click="squareClick(3)"
        :class="{selection: selectionSquare[3]}"
      ></div>
    </div>
    <div class="settings">
      <div v-show="gameState === 'won'">You won!</div>
      <div v-show="gameState === 'lost'">You lost!</div>
      <div>Round: {{ score }} / 10</div>
      <div class="levels">
        <span 
          class="level"
          :class="{level_active: level === 0}"
          @click="levelEasy"
        >Easy</span>
        <span 
          class="level"
          :class="{level_active: level === 1}"
          @click="levelMedium"
        >Medium</span>
        <span 
          class="level"
          :class="{level_active: level === 2}"
          @click="levelHard"
        >Hard</span>
      </div>
      <div>
        <span v-show="state === 'init'" @click="start">START</span>
        <span v-show="state === 'running'" @click="reset">RESET</span>
      </div>
    </div>
  </div>
</template>

<script>
import { Howl } from 'howler'
import ToggleButton from 'vue-js-toggle-button'

const timeInterval = [1500, 1000, 400]

export default {
  data() {
    return {
      error: false,
      victory: false,
      score: 0,
      level: 1,
      signal: [],
      signalInterval: null,
      repeatIndex: 0,
      state: 'init',
      gameState: null,
      sounds: [
        new Howl({ src: ['https://s3.amazonaws.com/freecodecamp/simonSound1.mp3'] }),
        new Howl({ src: ['https://s3.amazonaws.com/freecodecamp/simonSound2.mp3'] }),
        new Howl({ src: ['https://s3.amazonaws.com/freecodecamp/simonSound3.mp3'] }),
        new Howl({ src: ['https://s3.amazonaws.com/freecodecamp/simonSound4.mp3'] })
      ],
      selectionSquare: [false, false, false, false]
    }
  },
  methods: {
    start() {
      this.state = 'running'
      this.nextRound()
    },
    nextRound() {
      this.signal.push(Math.floor(Math.random() * 4))
      this.gameSignal()
    },
    gameSignal() {
      this.gameState = 'game_signal'
      let signalIndex = 0
      setTimeout(() => {
        this.signalInterval = setInterval(() => {
          this.clickingOnSquare(this.signal[signalIndex])
          signalIndex++
          if (signalIndex === this.signal.length) {
            clearInterval(this.signalInterval)
            this.repeatIndex = 0
            this.gameState = 'repeated_signal'
            return
          }
        }, timeInterval[this.level])
      }, 500)
    },
    clickingOnSquare(squareIndex) {
      this.$set(this.selectionSquare, squareIndex, true)
      this.sounds[squareIndex].play()
      setTimeout(() => {
        this.$set(this.selectionSquare, squareIndex, false)
      }, 300)
    },
    squareClick(squareIndex) {
      this.clickingOnSquare(squareIndex)
      if (this.signal[this.repeatIndex] != squareIndex) {
        this.error = true
        setTimeout(() => {
          this.error = false
        }, 500)
        this.gameState = 'lost'
        this.signal = []
        this.score = 0
        this.state = 'init'
        return
      }
      this.repeatIndex++
      if (this.repeatIndex === this.signal.length) {
        this.score++
        if (this.score === 10) {
          this.gameState = 'won'
          setTimeout(() => {
            this.victory = true
          }, 1000)
          return
        }
        this.nextRound()
      }
    },
    reset() {
      clearInterval(this.signalInterval)
      this.signal = []
      this.score = 0
      this.state = 'init'
      this.gameState = null
      this.victory = false
    },
    levelEasy() {
      this.level = 0
      this.reset()
    },
    levelMedium() {
      this.level = 1
      this.reset()
    },
    levelHard() {
      this.level = 2
      this.reset()
    }
  }
}
</script>

<style lang="scss">
.home {
  height: 100vh;
  display: grid;
  justify-content: center;
  align-content: center;
}
.squares {
  width: 200px;
  height: 200px;
  position: relative;
}
.square {
  width: 100px;
  height: 100px;
  opacity: 0.6;
  position: absolute;
  &.selection {
    opacity: 1;
  }
  &_blue {
    background-color: #1e90ff;
    bottom: 50%;
    right: 50%;
  }
  &_red {
    background-color: #FF5643;
    bottom: 50%;
    left: 50%;
  }
  &_yellow {
    background-color: #FEEF33;
    top: 50%;
    right: 50%;
  }
  &_green {
    background-color: #BEDE15;
    top: 50%;
    left: 50%;
  }
}
.settings {
  margin-top: 32px;
  display: grid;
  justify-content: center;
  text-align: center;
  > div {
    margin: 4px 0;
    &:last-child {
      span {
        cursor: pointer;
      }
    }
  }
}
.levels {
  .level {
    cursor: pointer;
    &:nth-child(2) {
      margin: 0 8px;
    }
    &_active {
      border-bottom: 1px solid #000000;
    }
  }
}
.error {
  animation: error 500ms linear forwards;
}

@keyframes error {
  0% {
    transform: rotate(-10deg);
  }
  25% {
    transform: rotate(10deg);
  }
  75% {
    transform: rotate(-10deg);
  }
  100% {
    transform: rotate(0deg);
  }
}

.victory {
  animation: victory 3000ms linear forwards;
}

@keyframes victory {
  1% {
    opacity: 1;
  }
  10% {
    opacity: 0;
  }
  20% {
    opacity: 1;
  }
  30% {
    opacity: 0;
  }
  40% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
  60% {
    opacity: 1;
  }
  70% {
    opacity: 0;
  }
  80% {
    opacity: 1;
  }
  90% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
</style>
