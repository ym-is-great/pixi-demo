<template>
  <div id="app">

    <!-- Loading -->
    <transition name="fade" mode="out-in">
      <div v-if="loading" class="loading">
        <p>{{ progress }}%</p>
      </div>
    </transition>

    <!-- Canvas 容器 -->
    <div ref="canvas-container"></div>

  </div>
</template>

<script>
import * as PIXI from 'pixi.js'
import { random } from 'lodash'

export default {
  name: 'app',
  computed: {
    tileLength () {
      return this.app.view.width / 4
    }
  },
  data () {
    return {
      clientWidth: null,
      clientHeight: null,
      loading: true,
      progress: 0,
      app: null,
      cat: null,
      ball: null,
      group: null
    }
  },
  mounted () {
    this.init()
    this.preload().then(() => {
      this.loading = false
      this.drawBackground()
      this.createCat()
      this.createSoccerBall()
      this.drawGroup()
      this.animateCat()
      this.animateSoccerBall()
      this.handleTouch()
    })
  },
  methods: {
    // 初始化
    init () {
      this.clientWidth = document.body.clientWidth
      this.clientHeight = document.body.clientHeight
      this.app = new PIXI.Application({
        width: this.clientWidth * 2,
        height: this.clientHeight * 2,
        // antialias: true
      })
      this.app.renderer.backgroundColor = 0x4caf50
      this.$refs['canvas-container'].appendChild(this.app.view)
    },
    // 预加载
    preload () {
      return new Promise((resolve, reject) => {
        PIXI.loader.add([
          require('@/assets/img/cat.png'),
          require('@/assets/img/map.png'),
          require('@/assets/img/soccer_ball.png')
        ])
        .on('progress', loader => {
          this.progress = parseInt(loader.progress)
        })
        .load(resolve)
      })
    },
    // 绘制背景
    drawBackground () {
      const texture = PIXI.loader.resources[require('@/assets/img/map.png')].texture
      texture.frame = new PIXI.Rectangle(199, 54, 48, 48)
      const sprite = new PIXI.extras.TilingSprite(texture, this.app.view.width, this.app.view.height)
      console.log('this.tileLength: ', this.tileLength)
      sprite.tileScale.set(this.tileLength / 48)
      sprite.tilePosition.x = 0
      sprite.tilePosition.y = 0
      this.app.stage.addChild(sprite)
    },
    // 创建猫咪精灵
    createCat () {
      const sprite = new PIXI.Sprite(
        PIXI.loader.resources[require('@/assets/img/cat.png')].texture
      )
      sprite.width = this.tileLength
      sprite.height = this.tileLength
      this.cat = sprite
    },
    // 创建足球精灵
    createSoccerBall () {
      const sprite = new PIXI.Sprite(
        PIXI.loader.resources[require('@/assets/img/soccer_ball.png')].texture
      )
      sprite.width = sprite.height = this.tileLength * 0.4
      this.ball = sprite
    },
    // 绘制分组（猫咪和足球）
    drawGroup () {
      const group = new PIXI.Container()
      group.addChild(this.cat)
      group.addChild(this.ball)
      this.app.stage.addChild(group)
      this.group = group
    },
    // 给猫咪添加动画
    animateCat () {
      const sprite = this.cat
      sprite.direction = 'left'
      this.app.ticker.add((delta) => {
        if (sprite.direction === 'left') {
          if (sprite.x > -2) sprite.x -= 0.1
          else sprite.direction = 'right'
        }
        if (sprite.direction === 'right') {
          if (sprite.x < 2) sprite.x += 0.1
          else sprite.direction = 'left'
        }
      })
    },
    // 给足球添加动画
    animateSoccerBall () {
      const sprite = this.ball
      sprite.anchor.set(0.5, 0.5)
      sprite.position.x = sprite.width / 2
      sprite.position.y = sprite.width / 2
      this.app.ticker.add((delta) => {
        sprite.rotation -= 0.02
      })
    },
    // 处理触摸事件
    handleTouch () {
      this.group.interactive = true
      this.group.on('tap', event => {
        const directions = []
        if (this.group.y > 0) directions.push('up')
        if (this.group.y < this.app.view.height - this.tileLength) directions.push('down')
        if (this.group.x > 0) directions.push('left')
        if (this.group.x < this.app.view.width - this.tileLength) directions.push('right')
        const direction = directions[random(0, directions.length - 1)]
        console.log('direction: ', direction)
        switch (direction) {
          case 'up':
            this.group.y -= this.tileLength
            break
          case 'down':
            this.group.y += this.tileLength
            break
          case 'left':
            this.group.x -= this.tileLength
            break
          case 'right':
            this.group.x += this.tileLength
        }
      })
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}
html, body, #app {
  width: 100%;
  height: 100%;
  overflow: hidden;
}
body {
  margin: 0;
}
canvas {
  transform-origin: top left;
  transform: scale(0.5);
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .3s ease-in-out;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
.loading {
  width: 100%;
  height: 100%;
  position: absolute;
  background: #4caf50;
  z-index: 1;
}
.loading p {
  width: 100%;
  margin: 0;
  color: #FFEB3B;
  text-shadow: 0.2rem 0.2rem 0 #000;
  text-align: center;
  font-size: 3.5rem;
  font-weight: bold;
  font-family: monospace;
  position: absolute;
  top: 45%;
  transform: translateY(-50%);
}
</style>
