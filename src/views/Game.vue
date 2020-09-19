<template>
  <div class="game" @mousedown="handleClick">
    <div
      class="player"
      :class="{
        falling: player.flapV < player.step / 10,
        rising: player.flapV > player.step / 3
      }"
      :style="{
        bottom: player.y + 'px',
        left: player.x + 'px',
        width: player.width + 'px',
        height: player.height + 'px'
      }"
      v-show="player.visible"
    />

    <div class="gates">
      <div
        v-for="gate in gates"
        class="gate"
        :key="gate.id"
        :style="{
          top: gate.y + 'px',
          left: gate.x + 'px',
          width: gate.width + 'px',
          height: gate.height + 'px'
        }"
      />
    </div>

    <div class="ground" />
    <div class="timer">
      {{ formatedGameTime }}
    </div>
    <!-- <div class="collision-switch" v-if="showCollision" /> -->
  </div>
</template>

<script>
export default {
  name: 'Game',
  data () {
    return {
      player: {
        x: 200,
        y: 0,
        v: 1,
        flapV: 0,
        step: 50,
        visible: false,
        width: 100,
        height: 70
      },
      gates: [],
      gatesTimeOut: 0,
      gravity: 10,
      gameInterval: 10,
      flapKeyLocked: false,
      gameTime: 0,
      gameStatus: false
    }
  },
  computed: {
    formatedGameTime () {
      return parseInt(this.gameTime / 1000)
    }
    // showCollision () {
    //   let isAnyCollision = false
    //   this.gates.forEach((gate, index) => {
    //     if (this.isCollision(gate)) {
    //       isAnyCollision = true
    //     }
    //   })
    //   return isAnyCollision
    // }
  },
  mounted () {
    window.addEventListener('keydown', this.handleKeydown)
    window.addEventListener('keyup', this.handleKeyup)
    this.initGame()
  },
  beforeUnmount () {
    window.removeEventListener('keydown', this.handleKeydown)
    window.removeEventListener('keyup', this.handleKeyup)
  },
  methods: {
    initGame () {
      this.gameTime = 0
      this.gameStatus = true
      this.gates = []
      if (this.gameInterval) clearInterval(this.gameInterval)
      this.playerReset()
      this.gameInterval = setInterval(this.gameLoop, 1000 / 60)
    },
    gameLoop () {
      // game time
      this.gameTime += 1000 / 60

      // gravity
      if (this.player.y >= 10) this.player.y -= this.gravity * this.player.v
      if (this.player.v < 1) this.player.v *= 1.1
      // touching the ground
      if (this.player.y < 10) this.gameOver()

      // adding vector to players position
      if (this.player.y >= 10) this.player.y += this.player.flapV
      if (this.player.flapV > 0) this.player.flapV *= 0.8
      if (this.player.flapV < 0.5) this.player.flapV = 0

      // flapping up to the ceiling
      if (this.player.y > window.innerHeight - this.player.height) {
        this.player.flapV = 0
        this.player.y = window.innerHeight - this.player.height
      }

      // gates
      if (this.formatedGameTime > 1 && this.gatesTimeOut <= 0) this.gatesProduction()
      if (this.gatesTimeOut > 0) this.gatesTimeOut -= 1000 / 60
      this.gatesMovement()
    },
    handleKeydown (e) {
      console.log(e)
      if (e.code === 'Space' || e.code === 'ArrowUp') {
        if (!this.flapKeyLocked) this.flap()
        this.flapKeyLocked = true
      }
      if (e.code === 'KeyR') {
        this.initGame()
      }
    },
    handleKeyup (e) {
      if (e.code === 'Space' || e.code === 'ArrowUp') this.flapKeyLocked = false
    },
    flap () {
      if (!this.gameStatus) return
      this.player.flapV += this.player.step
      this.player.v *= 0.5
    },
    handleClick () {
      if (this.gameStatus) {
        this.flap()
      } else {
        this.initGame()
      }
    },
    playerReset () {
      this.player.visible = true
      this.player.y = window.innerHeight / 2
      this.player.flapV = 0
    },
    gatesProduction () {
      const height = 320
      const newGate = {
        height,
        width: 180,
        x: window.innerWidth,
        y: (window.innerHeight - height) * Math.random()
      }
      this.gates.push(newGate)
      this.gatesTimeOut = 1500
    },
    gatesMovement () {
      this.gates.forEach((gate, index) => {
        gate.x -= 5
        // gate goes off the screen, it gets deleted
        if (gate.x + gate.width < 0) {
          this.gates.splice(index, 1)
        }
        // collision with player
        if (this.isCollision(gate)) {
          this.gameOver()
        }
      })
    },
    isCollision (gate) {
      const playerY = window.innerHeight - this.player.y - this.player.height
      const isCollision = (this.player.x + this.player.width) > gate.x &&
        (this.player.x + this.player.width) < (gate.x + gate.width) &&
        (playerY < gate.y || (playerY + this.player.height) > (gate.y + gate.height))
      return isCollision
    },
    gameOver () {
      clearInterval(this.gameInterval)
      this.gameStatus = false
    }
  }
}
</script>

<style lang="scss" scoped>
.game {
  background: rgb(84, 158, 255);
  height: 100vh;
}

.player {
  background-image: url('../assets/img/flappy-bird.png');
  background-repeat: no-repeat;
  background-position: center;
  background-size: contain;
  position: absolute;
  transition: all .05s ease;
  z-index: 2;

  &.falling {
    transform: rotateZ(10deg);
  }
  &.rising {
    transform: rotateZ(-10deg);
  }
}

.gates {
  .gate {
    position: absolute;

    &:before, &:after {
      background: transparent;
      background: linear-gradient(90deg, transparent 0%, rgba(58,118,52,0) 4%, rgba(9,84,1,1) 5%, rgba(89,213,76,1) 20%, rgba(138,246,126,1) 40%, rgba(138,246,126,1) 60%, rgba(89,213,76,1) 80%, rgba(9,84,1,1) 95%, rgba(255,255,255,0) 96%, transparent 100%);
    }

    &:before {
      border-bottom: 20px solid;
      border-image-slice: 1;
      border-image-source: linear-gradient(to left, rgb(7, 70, 0),rgb(71, 168, 61), rgb(113, 202, 103), rgb(71, 168, 61), rgb(7, 70, 0));
      content: '';
      position: absolute;
      bottom: calc(100% - 10px);
      height: 100vh;
      width: 100%;
    }

    &:after {
      border-top: 20px solid;
      border-image-slice: 1;
      border-image-source: linear-gradient(to left, rgb(7, 70, 0),rgb(71, 168, 61), rgb(113, 202, 103), rgb(71, 168, 61), rgb(7, 70, 0));
      content: '';
      position: absolute;
      top: calc(100% - 10px);
      height: 100vh;
      width: 100%;
    }
  }
}

.ground {
  background: green;
  position: absolute;
  left: 0;
  bottom: 0;
  right: 0;
  height: 10px;
}

.timer {
  background: #000;
  color: #ffffff;
  position: absolute;
  right: 10px;
  bottom: 20px;
  padding: 10px 15px;
  font-size: 20px;
}

.collision-switch {
  background: red;
  position: absolute;
  left: 10px;
  bottom: 20px;
  width: 30px;
  height: 30px;
}
</style>
