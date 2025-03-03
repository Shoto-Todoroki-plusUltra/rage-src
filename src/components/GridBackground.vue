<template>
  <div class="grid-container">
    <div class="grid-particles">
      <div v-for="(particle, index) in particles" 
           :key="index" 
           class="grid-particle"
           :style="particle.style">{{ particle.emoji }}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'GridBackground',
  data() {
    return {
      particles: [],
      emojis: ['🎀', '💖', '✨', '💅', '👑', '💕', '💄', '😩', '😌', '🌷'],
      maxParticles: 100
    }
  },
  mounted() {
    this.generateParticles()
  },
  methods: {
    generateParticles() {
      this.particles = Array.from({ length: this.maxParticles }, (_, i) => {
        const randomEmoji = this.emojis[Math.floor(Math.random() * this.emojis.length)]
        return {
          emoji: randomEmoji,
          style: {
            left: `${Math.random() * 100}%`,
            top: `${Math.random() * 100}%`,
            animationDelay: `${i * 0.1}s`,
            opacity: Math.random() * 0.5 + 0.2,
            fontSize: `${Math.random() * 14 + 12}px`,
            filter: `drop-shadow(0 0 4px #FF69B4)`,
            animationDuration: `${Math.random() * 4 + 2}s`
          }
        }
      })
    },
    createParticle(left, top, delay) {
      const randomEmoji = this.emojis[Math.floor(Math.random() * this.emojis.length)]
      return {
        emoji: randomEmoji,
        style: {
          left: left,
          top: top,
          animationDelay: delay,
          opacity: Math.random() * 0.5 + 0.2,
          fontSize: `${Math.random() * 14 + 12}px`,
          filter: `drop-shadow(0 0 4px #FF69B4)`,
          animationDuration: `${Math.random() * 4 + 2}s`
        }
      }
    },
    addParticleAtPosition(x, y) {
      const container = document.querySelector('.grid-container')
      const percentX = (x / container.offsetWidth) * 100
      const percentY = (y / container.offsetHeight) * 100
      
      const newParticle = this.createParticle(
        `${percentX}%`, 
        `${percentY}%`,
        '0s'
      )
      newParticle.style.animation = 'particleSpawn 0.5s ease-out, particleGlow ease-in-out infinite';
      newParticle.style.animationDelay = '0s, 0.5s';
      this.particles.push(newParticle)

      if (this.particles.length > this.maxParticles + 50) {
        this.particles.shift() 
      }
    }
  }
}
</script>

<style scoped>
.grid-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  overflow: hidden;
}

.grid-particles {
  position: absolute;
  width: 100%;
  height: 100%;
}

.grid-particle {
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  animation: particleGlow ease-in-out infinite;
  user-select: none;
}

@keyframes particleGlow {
  0%, 100% { 
    transform: scale(0.8) rotate(0deg);
    opacity: 0.2;
    filter: drop-shadow(0 0 3px #FF69B4);
  }
  50% { 
    transform: scale(1.2) rotate(10deg);
    opacity: 0.6;
    filter: drop-shadow(0 0 6px #FF1493);
  }
}
</style>
