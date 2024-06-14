<template>
  <div class="container">
    <Score :level="level" :score="score" :life="life" />
    <canvas ref="canvas" width="800" height="600"></canvas>
    <GameOver :gameOver="gameOver" />
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import GameOver from './GameOver.vue'
import Score from './Score.vue'

const canvas = ref(null)
let ctx = null

let score = ref(0)
let level = ref(0)
let life = ref(5)
let gameOver = ref(false)

const playerImage = new Image()
const enemyImage = new Image()
const asteroidImage = new Image()
playerImage.src = '/assets/player.png'
enemyImage.src = '/assets/enemy.png'
asteroidImage.src = '/assets/asteroid.png'

// Ensure images are loaded before starting the game
let imagesLoaded = 0
playerImage.onload = () => checkImagesLoaded()
enemyImage.onload = () => checkImagesLoaded()
asteroidImage.onload = () => checkImagesLoaded()
playerImage.onerror = () => console.error('Failed to load player image')
enemyImage.onerror = () => console.error('Failed to load enemy image')
asteroidImage.onerror = () => console.error('Failed to load enemy image')

function checkImagesLoaded() {
  imagesLoaded++
  if (imagesLoaded === 3) {
    startGame()
  }
}

onMounted(() => {
  ctx = canvas.value.getContext('2d')
  startGame()
})

class Particle {
  constructor({ position, velocity, radius, color }) {
    this.position = position
    this.velocity = velocity
    this.radius = radius
    this.color = color
    this.opacity = 1
  }

  draw() {
    ctx.save()
    ctx.globalAlpha = this.opacity
    ctx.beginPath()
    ctx.arc(this.position.x, this.position.y, this.radius, 0, Math.PI * 2)
    ctx.fillStyle = this.color
    ctx.fill()
    ctx.closePath()
    ctx.restore()
  }

  update() {
    this.draw()
    this.position.x += this.velocity.x
    this.position.y += this.velocity.y
    this.opacity -= 0.01
  }
}

class Bullet {
  constructor({ position, velocity, color }) {
    this.position = position
    this.velocity = velocity
    this.color = color
  }

  draw() {
    ctx.fillStyle = this.color
    ctx.fillRect(this.position.x, this.position.y, this.position.width, this.position.height)
  }

  update() {
    this.draw()
    this.position.y += this.velocity.y
  }
}

function startGame() {
  // Initialize game variables
  let keysPressed = {}
  const player = { x: 400, y: 550, width: 50, height: 50 }
  const bullets = []
  const enemyBullets = []
  let enemies = []
  let asteroids = []
  const particles = []

  function getRandomInt(max) {
    const randomNumber = Math.floor(Math.random() * max)
    return randomNumber > 0 ? randomNumber : getRandomInt(max)
  }

  function spawnEnemies() {
    for (let i = 0; i <= level.value; i++)
      enemies.push({
        x: getRandomInt(10) * 60,
        y: getRandomInt(10),
        width: 50,
        height: 50
      })
  }

  function spawnAsteroids() {
    for (let i = 0; i <= level.value; i++)
      asteroids.push({
        x: getRandomInt(10) * 60,
        y: getRandomInt(10),
        width: 50,
        height: 50
      })
  }
  setInterval(spawnEnemies, 2000)
  setInterval(spawnAsteroids, 3000)

  function update() {
    // Clear the canvas
    ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)

    // Draw the player
    ctx.drawImage(playerImage, player.x, player.y, player.width, player.height)

    // Draw bullets
    bullets.forEach((bullet, index) => {
      if (bullet.y < 0) bullets.splice(index, 1)
      bullet.update()
    })

    particles.forEach((particle, particleIndex) => {
      particle.opacity <= 0 ? particles.splice(particleIndex, 1) : particle.update()
    })

    // Draw enemies
    if (enemies.length)
      enemies.forEach((enemy) => {
        enemy.y += 1
        ctx.drawImage(enemyImage, enemy.x, enemy.y, enemy.width, enemy.height)
      })

    if (asteroids.length)
      asteroids.forEach((asteroid) => {
        asteroid.y += 1
        ctx.drawImage(asteroidImage, asteroid.x, asteroid.y, asteroid.width, asteroid.height)
      })
    // Collision detection
    if (bullets.length)
      bullets.forEach((bullet, bulletIndex) => {
        enemies.forEach((enemy, enemyIndex) => {
          if (
            bullet.position.x < enemy.x + enemy.width &&
            bullet.position.x + bullet.position.width > enemy.x &&
            bullet.position.y < enemy.y + enemy.height &&
            bullet.position.height + bullet.position.y > enemy.y
          ) {
            for (let i = 0; i < 15; i++) {
              const particle = new Particle({
                position: { x: enemy.x + enemy.width / 2, y: enemy.y + enemy.height / 2 },
                velocity: { x: Math.random(5) - 0.5, y: Math.random(5) - 0.5 },
                radius: 5,
                color: '#B4382B'
              })

              particles.push(particle)
            }
            enemies.splice(enemyIndex, 1)
            bullets.splice(bulletIndex, 1)
            score.value += 10
            level.value = score.value / 100
          }
        })
        asteroids.forEach((asteroid, asteroidIndex) => {
          if (
            bullet.position.x < asteroid.x + asteroid.width &&
            bullet.position.x + bullet.position.width > asteroid.x &&
            bullet.position.y < asteroid.y + asteroid.height &&
            bullet.position.height + bullet.position.y > asteroid.y
          ) {
            for (let i = 0; i < 15; i++) {
              const particle = new Particle({
                position: {
                  x: asteroid.x + asteroid.width / 2,
                  y: asteroid.y + asteroid.height / 2
                },
                velocity: { x: Math.random(5) - 0.5, y: Math.random(5) - 0.5 },
                radius: 5,
                color: 'gray'
              })

              particles.push(particle)
            }
            asteroids.splice(asteroidIndex, 1)
            bullets.splice(bulletIndex, 1)
            score.value += 10
            level.value = score.value / 100
          }
        })
      })

    // Game over condition
    enemies.forEach((enemy, enemyIndex) => {
      if (enemy.y + enemy.height > player.y) {
        life.value -= 1
        enemies.splice(enemyIndex, 1)
      }
    })

    if (life.value < 1) {
      gameOver.value = true
      return
    }
    requestAnimationFrame(update)
  }

  function handleKeyDown(event) {
    keysPressed[event.key] = true
    if (gameOver.value) {
      gameOver.value = false
      life.value = 5
      startGame()
      return
    }
    Object.keys(keysPressed).map((key) => {
      switch (key) {
        case 'ArrowLeft':
          player.x = Math.max(0, player.x - 15)
          break
        case 'ArrowRight':
          player.x = Math.min(canvas.value.width - player.width, player.x + 15)
          break
        case ' ':
          bullets.push(
            new Bullet({
              position: {
                x: player.x + player.width / 2 - 2.5,
                y: player.y,
                width: 5,
                height: 10
              },
              velocity: { y: -5 },
              color: 'yellow'
            })
          )
          break
      }
    })
  }

  window.addEventListener('keydown', handleKeyDown)
  window.addEventListener('keyup', (event) => {
    delete keysPressed[event.key]
  })

  update()
}
</script>

<style scoped>
@import url('../assets//main.css');
.container {
  position: relative;
}
canvas {
  background-color: var(--color-background);
  border: 1px solid #fff;
}
</style>
