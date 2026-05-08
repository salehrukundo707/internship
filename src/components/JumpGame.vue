<template>
  <div class="game-container">
    <div class="score-board">Score: {{ score }}</div>
    
    <!-- Player and Obstacle -->
    <div class="player" :style="{ bottom: playerY + 'px' }"></div>
    <div class="obstacle" :style="{ left: obstaclePosition.x + 'px' }"></div>

    <div v-if="gameOver" class="game-over-overlay">
      <h1>GAME OVER</h1>
      <button @click="resetGame">Restart</button>
    </div>
  </div>
</template>

<script>
// Import your audio files from src/assets
import jumpSfx from '@/assets/jump.mp3';
import pointsSfx from '@/assets/points.mp3';
import backMusic from '@/assets/back.mp3';

export default {
  data() {
    return {
      score: 0,
      gameOver: false,
      obstaclePosition: { x: 500 },
      playerY: 0,
      isJumping: false,
      animationFrame: null,
      // Audio Objects
      sounds: {
        jump: new Audio(jumpSfx),
        points: new Audio(pointsSfx),
        bgm: new Audio(backMusic)
      }
    };
  },
  mounted() {
    // Setup background music settings
    this.sounds.bgm.loop = true;
    this.sounds.bgm.volume = 1;

    this.gameLoop();

    // Spacebar listener
    window.addEventListener('keydown', (e) => {
      if (e.code === 'Space') this.jump();
    });
  },
  methods: {
    gameLoop() {
      if (this.gameOver) return;

      this.obstaclePosition.x -= 5;

      if (this.obstaclePosition.x < -40) {
        this.obstaclePosition.x = 500;
        this.incrementScore();
      }

      this.checkCollision();
      this.animationFrame = requestAnimationFrame(this.gameLoop);
    },

    jump() {
      if (this.isJumping || this.gameOver) return;

      // Play Jump Sound
      this.sounds.jump.currentTime = 0;
      this.sounds.jump.play();

      // Start BGM on first interaction (required by most browsers)
      if (this.sounds.bgm.paused) {
        this.sounds.bgm.play();
      }

      this.isJumping = true;
      let height = 0;
      
      const jumpInterval = setInterval(() => {
        if (height >= 150) {
          clearInterval(jumpInterval);
          const fallInterval = setInterval(() => {
            if (height <= 0) {
              clearInterval(fallInterval);
              this.isJumping = false;
            }
            height -= 5;
            this.playerY = height;
          }, 20);
        }
        height += 5;
        this.playerY = height;
      }, 20);
    },

    checkCollision() {
      const playerRect = { 
        left: 50, 
        right: 90, 
        top: this.playerY + 40, 
        bottom: this.playerY 
      };
      const obsRect = { 
        left: this.obstaclePosition.x, 
        right: this.obstaclePosition.x + 40, 
        top: 40, 
        bottom: 0 
      };

      if (
        playerRect.right > obsRect.left &&
        playerRect.left < obsRect.right &&
        playerRect.bottom < obsRect.top
      ) {
        this.gameOver = true;
        
        // Stop music on game over
        this.sounds.bgm.pause();
        
        cancelAnimationFrame(this.animationFrame);
      }
    },

    incrementScore() {
      this.score++;
      // Play Points Sound
      this.sounds.points.currentTime = 0;
      this.sounds.points.play();
    },

    resetGame() {
      this.score = 0;
      this.gameOver = false;
      this.obstaclePosition.x = 500;
      this.playerY = 0;

      // Restart Music
      this.sounds.bgm.currentTime = 0;
      this.sounds.bgm.play();

      this.gameLoop();
    }
  },
  // Clean up sounds and listeners if component is destroyed
  beforeUnmount() {
    this.sounds.bgm.pause();
    window.removeEventListener('keydown', this.jump);
  }
};
</script>

<style scoped>
.game-container {
  width: 500px;
  height: 200px;
  border-bottom: 2px solid black;
  margin: 50px auto;
  position: relative;
  overflow: hidden;
  background: #f0f0f0;
}
.player {
  width: 40px;
  height: 40px;
  background: blue;
  position: absolute;
  left: 50px;
}
.obstacle {
  width: 40px;
  height: 40px;
  background: red;
  position: absolute;
  bottom: 0;
}
.score-board {
  position: absolute;
  top: 10px;
  right: 10px;
  font-family: Arial, sans-serif;
  font-weight: bold;
}
.game-over-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.7);
  color: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  background: #4caf50;
  color: white;
  border: none;
  border-radius: 5px;
}
button:hover {
  background: #45a049;
}
</style>
