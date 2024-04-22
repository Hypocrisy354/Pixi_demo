<template>
  <div></div>
</template>

<script setup>
// 导入pixi.js
// import { DropShadowFilter } from 'pixi-filters'  // 滤镜
import {
  Container,
  Application,
  Assets,
  Text,
  Texture,
  Rectangle,
  Sprite,
  AnimatedSprite,
  TilingSprite
} from 'pixi.js'

const init = async () => {
  // 创建应用
  const app = new Application()
  await app.init({
    width: window.innerWidth,
    height: window.innerHeight,
    backgroundColor: 0xffffff,
    resolution: window.devicePixelRatio || 1
  })

  // 将应用画布添加到DOM中
  document.body.appendChild(app.canvas)

  // 创建容器
  const container = new Container()

  // 将容器添加到舞台
  app.stage.addChild(container)

  // 添加游戏精灵纹理
  const baseTexture = await Assets.load(require('../public/images/game.png'))

  // 创建恐龙纹理
  const dinoTexture = new Texture({
    source: baseTexture,
    frame: new Rectangle(75, 0, 88, 100)
  })

  // 创建恐龙精灵
  const dinoSprite = new Sprite(dinoTexture)
  dinoSprite.visible = false
  container.addChild(dinoSprite)
  // 恐龙跑步动画

  const runTextures = []
  for (let i = 0; i < 2; i++) {
    runTextures.push(
      new Texture({
        source: baseTexture,
        frame: new Rectangle(1680 + (2 + i) * 88, 0, 82, 100)
      })
    )
  }
  // console.log(runTextures)
  const runAnimation = new AnimatedSprite(runTextures)
  runAnimation.x = 60
  runAnimation.y = window.innerHeight - 50 - 100
  runAnimation.animationSpeed = 0.1
  runAnimation.play()
  // runAnimation.visible = false
  container.addChild(runAnimation)

  // 恐龙跳跃精灵
  const jumpTexture = new Texture({
    source: baseTexture,
    frame: new Rectangle(1680, 0, 82, 100)
  })
  const jumpSprite = new Sprite(jumpTexture)
  jumpSprite.x = 60
  jumpSprite.y = window.innerHeight - 50 - 100
  jumpSprite.visible = false
  container.addChild(jumpSprite)

  // 创建地面精灵
  const groundTexture = new Texture({
    source: baseTexture,
    frame: new Rectangle(50, 100, 2300, 30)
  })
  const groundSprite = new TilingSprite(groundTexture)
  groundSprite.width = window.innerWidth
  groundSprite.height = 30

  // 设置地面精灵位置
  groundSprite.position.set(0, window.innerHeight - 50)
  container.addChild(groundSprite)

  // 创建仙人掌精灵
  const cactusTexture = new Texture({
    source: baseTexture,
    frame: new Rectangle(515, 0, 30, 60)
  })
  const cactusSprite = new Sprite(cactusTexture)
  cactusSprite.x = window.innerWidth / 2
  cactusSprite.y = window.innerHeight - 50 - 60
  cactusSprite.visible = false
  const cactusSprite1 = new Sprite(cactusTexture)
  cactusSprite1.x = window.innerWidth / 2
  cactusSprite1.y = window.innerHeight - 280
  // cactusSprite.visible = false
  container.addChild(cactusSprite, cactusSprite1)
  // 创建云朵精灵
  const cloudTexture = new Texture({
    source: baseTexture,
    frame: new Rectangle(172, 0, 88, 60)
  })
  const cloudSprite1 = new Sprite(cloudTexture)
  cloudSprite1.x = window.innerWidth / 1.5
  cloudSprite1.y = window.innerHeight / 4
  const cloudSprite2 = new Sprite(cloudTexture)
  cloudSprite2.x = window.innerWidth / 2.5
  cloudSprite2.y = window.innerHeight / 5
  container.addChild(cloudSprite1, cloudSprite2)

  // 创建文字
  const startText = new Text({
    text: '开始游戏',
    style: {
      fontFamily: 'Arial',
      fontSize: 36,
      fill: 0x333,
      align: 'center'
    }
  })
  startText.x = window.innerWidth / 2
  startText.y = window.innerHeight / 2
  startText.anchor.set(0.5)
  container.addChild(startText)
  startText.interactive = true
  startText.on('click', () => {
    playGame()
  })
  // 游戏状态
  let isGaming = false
  let isGameOver = false

  // 开始游戏 空格跳跃
  function playGame() {
    isGaming = true
    console.log('开始游戏')
    // 显示恐龙跑步动画
    runAnimation.visible = true
    runAnimation.x = 60
    runAnimation.y = window.innerHeight - 50 - 100
  }
  // 游戏得分
  let score = 0
  // 初始化跳跃速度
  let jumpVelocity = 20
  // 初始化重力
  let gravity = 1
  // 游戏循环
  app.ticker.add(delta => {
    if (isGameOver) return
    // 地面移动
    groundSprite.tilePosition.x -= 10
    // 云朵移动
    cloudSprite1.x -= 10
    cloudSprite2.x -= 10
    // 背景仙人掌移动
    cactusSprite1.x -= 10
    //重置
    if (cactusSprite1.x < -30) {
      cactusSprite1.x = window.innerWidth + window.innerWidth * Math.random()
    }
    if (cloudSprite1.x < -88) {
      cloudSprite1.x = window.innerWidth + window.innerWidth * Math.random()
    }
    if (cloudSprite2.x < -88) {
      cloudSprite2.x = window.innerWidth + window.innerWidth * Math.random()
    }
    if (isGaming) {
      startText.visible = false
      cactusSprite.visible = true
      cactusSprite1.visible = false
      // 仙人掌移动
      cactusSprite.x -= 10
      // 是否跳跃仙人掌
      if (cactusSprite.x < -30) {
        cactusSprite.x =
          window.innerWidth + window.innerWidth * Math.random() * 0.1
        score++
      }
      // 跳跃设置
      if (jumpSprite.visible) {
        jumpVelocity -= gravity
        jumpSprite.y -= jumpVelocity
        if (jumpSprite.y > window.innerHeight - 50 - 100) {
          jumpSprite.y = window.innerHeight - 50 - 100
          jumpVelocity = 20
          jumpSprite.visible = false
          runAnimation.visible = true
        }
      }
      // 碰撞检测
      if (
        jumpSprite.y > cactusSprite.y - 60 &&
        cactusSprite.x < jumpSprite.x + 60 &&
        cactusSprite.x > jumpSprite.x - 60
      ) {
        // 游戏结束
        gameOver()
        // 显示得分
        let overText = new Text({
          text: `游戏结束,最后得分:${score}\n点击刷新`,
          style: {
            fontFamily: 'Arial',
            fontSize: 36,
            fill: 0x333,
            align: 'center'
          }
        })
        overText.x = window.innerWidth / 2
        overText.y = window.innerHeight / 2
        overText.anchor.set(0.5)
        container.addChild(overText)
        overText.interactive = true
        overText.on('click', () => {
          location.reload()
        })
      }
    }
  })
  function gameOver() {
    console.log('游戏结束')
    isGameOver = true
  }
  // 监听空格键
  window.addEventListener('keydown', e => {
    if (e.code === 'Space') {
      console.log('跳跃')
      runAnimation.visible = false
      jumpSprite.visible = true
      jumpVelocity = 20
    }
  })
}
init()
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
canvas {
  width: 100vw;
  height: 100vh;
  position: fixed;
  left: 0;
  top: 0;
}
</style>
