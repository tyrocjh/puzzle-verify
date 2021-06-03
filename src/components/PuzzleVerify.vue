<template>
  <div class="puzzle-verify" :style="{ padding: boxPadding+'px' }">
    <header class="action-box">
      <div class="descript">拖动下方滑块完成拼图</div>
      <div class="actions">
        <img :src="refreshImg" @click="handleRefresh" />
        <img v-if="showCloseBtn" class="close-btn" :src="closeImg" @click="handleClose" />
      </div>
    </header>
    <section class="puzzle-box" :style="{ width: imgWidth+'px', height: imgHeight+'px' }">
      <img :src="imgSrc" />
      <canvas id="puzzle-box" ref="puzzleBox" :width="imgWidth" :height="imgHeight"></canvas>

      <div class="puzzle-box-lost" :style="{left: offsetLeft+'px'}">
        <canvas ref="puzzleBoxLostShadow" id="puzzle-box-lost-shadow" :width="imgWidth" :height="imgHeight"></canvas>
        <canvas ref="puzzleBoxLost" id="puzzle-box-lost" :width="imgWidth" :height="imgHeight"></canvas>
      </div>

      <div :class="['tips', showTips?'show-tips':'']">
        <p class="success" v-if="verifyPass">验证通过</p>
        <p class="fail" v-else>验证失败</p>
      </div>
    </section>
    <footer class="slider-box">
      <div class="slider-bar"></div>
      <div ref="sliderButton" class="slider-button" @mousedown="startMove" @touchstart="startMove"><span></span></div>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'PuzzleVerify',
  props: {
    imgList: {
      // 图片列表
      type: Array,
      default: () => [require('@/assets/cat.jpg')]
    },
    imgWidth: {
      // 图片宽度
      type: Number,
      default: 300
    },
    imgHeight: {
      // 图片高度
      type: Number,
      default: 200
    },
    boxPadding: {
      // 盒子内边距
      type: Number,
      default: 20
    },
    puzzleType: {
      // 拼图类型
      type: String,
      default: 'square' // square puzzle
    },
    puzzleSize: {
      // 拼图大小
      type: Number,
      default: 40
    },
    puzzleRadius: {
      // 拼图圆角
      type: Number,
      default: 10
    },
    defaultOffsetLeft: {
      // 拼图偏移量
      type: Number,
      default: 10      
    },
    deviation: {
      // 误差范围
      type: Number,
      default: 5
    },
    showCloseBtn: {
      // 关闭按钮
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      showTips: false,
      verifyPass: false,
      refreshImg: require('@/assets/icon/refresh.png'),
      closeImg: require('@/assets/icon/close.png'),
      imgSrc: '',
      randomX: '',
      randomY: '',
      offsetLeft: '',
      moveStart: '',
      moveLeft: ''
    }
  },
  mounted() {
    this.setImage()
    this.initCanvas()
  },
  methods: {
    // 设置图片
    setImage() {
      let num = this.randomIntegerInRange(1, this.imgList.length)
      this.imgSrc = this.imgList[num - 1]
    },
    // 初始化画布
    initCanvas() {
      this.clearCanvas()
      this.drawCanvas()
    },
    // 画canvas
    drawCanvas() {
      this.calculate()

      let x = this.randomX, y = this.randomY, w = this.puzzleSize, h = this.puzzleSize, r = this.puzzleRadius
      this.drawPuzzleBox(x, y, w, h, r)
      this.drawPuzzleBoxLost(x, y, w, h, r)
      this.drawPuzzleBoxLostShadow(x, y, w, h, r)
    },
    // 计算位置
    calculate() {
      this.randomX = this.randomIntegerInRange(this.imgWidth / 3, this.imgWidth - this.puzzleSize * 1.5)
      this.randomY = this.randomIntegerInRange(this.puzzleSize, this.imgHeight - this.puzzleSize * 1.5)
      this.offsetLeft = -this.randomX + this.defaultOffsetLeft
    },
    // 画拼图
    drawPuzzleBox(x, y, w, h, r) {
      let puzzleBox = this.$refs.puzzleBox,
        ctx = puzzleBox.getContext('2d')
      
			ctx.globalCompositeOperation = 'xor'
			ctx.shadowBlur = 10
			ctx.shadowColor = '#fff'
			ctx.shadowOffsetX = 3
			ctx.shadowOffsetY = 3
			ctx.fillStyle = 'rgba(0, 0, 0, 0.7)'
      ctx.strokeStyle = 'rgba(0, 0, 0, 0)'
			ctx.lineWidth = 1
      
      ctx.beginPath()
      if (this.puzzleType === 'square') {
        ctx.moveTo(x + r, y)
        ctx.arcTo(x + w, y, x + w, y + h, r)
        ctx.arcTo(x + w, y + h, x, y + h, r)
        ctx.arcTo(x, y + h, x, y, r)
        ctx.arcTo(x, y, x + w, y, r)
      } else {
        w = w / 3
        ctx.moveTo(x, y)
        ctx.lineTo(x + w, y)
        ctx.bezierCurveTo(x + w, y - w, x + 2 * w, y - w, x + 2 * w, y)
        ctx.lineTo(x + 3 * w, y)
        ctx.lineTo(x + 3 * w, y + w)
        ctx.bezierCurveTo(x + 2 * w, y + w, x + 2 * w, y + 2 * w, x + 3 * w, y + 2 * w)
        ctx.lineTo(x + 3 * w, y + 3 * w)
        ctx.lineTo(x, y + 3 * w)
      }
      ctx.closePath()
      ctx.stroke()
      ctx.fill()
    },
    // 画缺失的拼图
    drawPuzzleBoxLost(x, y, w, h, r) {
      let puzzleBoxLost = this.$refs.puzzleBoxLost,
        ctx = puzzleBoxLost.getContext('2d')

      let image = new Image()
      image.src = this.imgSrc
      image.onload = () => {
        ctx.drawImage(image, 0, 0, this.imgWidth, this.imgHeight)
      }
      
      ctx.strokeStyle = 'rgba(0, 0, 0, 0)'

      ctx.beginPath()
      if (this.puzzleType === 'square') {
        ctx.moveTo(x + r, y)
        ctx.arcTo(x + w, y, x + w, y + h, r)
        ctx.arcTo(x + w, y + h, x, y + h, r)
        ctx.arcTo(x, y + h, x, y, r)
        ctx.arcTo(x, y, x + w, y, r)
      } else {
        w = w / 3
        ctx.moveTo(x, y)
        ctx.lineTo(x + w, y)
        ctx.bezierCurveTo(x + w, y - w, x + 2 * w, y - w, x + 2 * w, y)
        ctx.lineTo(x + 3 * w, y)
        ctx.lineTo(x + 3 * w, y + w)
        ctx.bezierCurveTo(x + 2 * w, y + w, x + 2 * w, y + 2 * w, x + 3 * w, y + 2 * w)
        ctx.lineTo(x + 3 * w, y + 3 * w)
        ctx.lineTo(x, y + 3 * w)
      }
      ctx.closePath()
      ctx.clip()
    },
    // 画缺失的拼图阴影
    drawPuzzleBoxLostShadow(x, y, w, h, r) {
      let puzzleBoxLostShadow = this.$refs.puzzleBoxLostShadow,
        ctx = puzzleBoxLostShadow.getContext('2d')

			ctx.shadowBlur = 20
      ctx.shadowColor = '#000'
      ctx.fillStyle = 'rgba(0, 0, 0, 1)'

      ctx.beginPath()
      if (this.puzzleType === 'square') {
        ctx.moveTo(x + r, y)
        ctx.arcTo(x + w, y, x + w, y + h, r)
        ctx.arcTo(x + w, y + h, x, y + h, r)
        ctx.arcTo(x, y + h, x, y, r)
        ctx.arcTo(x, y, x + w, y, r)
      } else {
        w = w / 3
        ctx.moveTo(x, y)
        ctx.lineTo(x + w, y)
        ctx.bezierCurveTo(x + w, y - w, x + 2 * w, y - w, x + 2 * w, y)
        ctx.lineTo(x + 3 * w, y)
        ctx.lineTo(x + 3 * w, y + w)
        ctx.bezierCurveTo(x + 2 * w, y + w, x + 2 * w, y + 2 * w, x + 3 * w, y + 2 * w)
        ctx.lineTo(x + 3 * w, y + 3 * w)
        ctx.lineTo(x, y + 3 * w)
      }
      ctx.closePath()
      ctx.fill()
    },
    // 清空画布
    clearCanvas() {
      const puzzleBox = this.$refs.puzzleBox,
        puzzleBoxLost = this.$refs.puzzleBoxLost,
        puzzleBoxLostShadow = this.$refs.puzzleBoxLostShadow

      puzzleBox.setAttribute('height', puzzleBox.getAttribute('height'));
      puzzleBoxLost.setAttribute('height', puzzleBoxLost.getAttribute('height'));
      puzzleBoxLostShadow.setAttribute('height', puzzleBoxLostShadow.getAttribute('height'));
    },
    // 指定范围内随机整数
    randomIntegerInRange(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min
    },
    // 触发移动
    startMove(e) {
      e = e || window.event
      this.moveStart = e.pageX || e.targetTouches[0].pageX
      this.addMouseMoveListener()
    },
    // 移动中
    moving(e) {
      e = e || window.event
      let moveX = e.pageX || e.targetTouches[0].pageX
      let left = moveX - this.moveStart
      
      if (left < 0 || left > this.imgWidth - this.puzzleSize - this.boxPadding) return

      this.moveLeft = left
      this.$refs.puzzleBoxLost.style.left = left + 'px'
      this.$refs.puzzleBoxLost.style.transition = 'inherit'
      this.$refs.puzzleBoxLostShadow.style.left = left + 'px'
      this.$refs.puzzleBoxLostShadow.style.transition = 'inherit'
      this.$refs.sliderButton.style.left = left + 5 + 'px'
      this.$refs.sliderButton.style.transition = 'inherit'
    },
    // 移动结束
    moveEnd() {
      this.removeMouseMoveListener()

      let minLeft = this.moveLeft + this.defaultOffsetLeft - this.deviation,
        maxLeft = this.moveLeft + this.defaultOffsetLeft + this.deviation
      
      this.showTips = true
      if (minLeft <= this.randomX && maxLeft >= this.randomX) {
        this.verifyPass = true
      } else {
        this.verifyPass = false
      }

      // 滑块复位
      setTimeout(() => {
        this.moveLeft = 0
        this.$refs.puzzleBoxLost.style.left = this.moveLeft + 'px'
        this.$refs.puzzleBoxLost.style.transition = 'left 0.5s'
        this.$refs.puzzleBoxLostShadow.style.left = this.moveLeft + 'px'
        this.$refs.puzzleBoxLostShadow.style.transition = 'left 0.5s'
        this.$refs.sliderButton.style.left = this.moveLeft + 5 + 'px'
        this.$refs.sliderButton.style.transition = 'left 0.5s'
      }, 400);

      // 成功/失败回调
      setTimeout(() => {
        if (this.verifyPass) {
          this.$emit('handle-success')
        } else {
          this.$emit('handle-error')
        }
        this.showTips = false
        this.initCanvas()
      }, 800);
    },
    // 点击刷新
    handleRefresh() {
      this.setImage()
      this.initCanvas()
    },
    // 点击关闭
    handleClose() {
      this.$emit('handle-close')
    },
    // 移动事件监听绑定
    addMouseMoveListener() {
			document.addEventListener('mousemove', this.moving)
			document.addEventListener('touchmove', this.moving)
			document.addEventListener('mouseup', this.moveEnd)
			document.addEventListener('touchend', this.moveEnd)
    },
    // 移动事件监听解绑
    removeMouseMoveListener() {
			document.removeEventListener('mousemove', this.moving)
			document.removeEventListener('touchmove', this.moving)
			document.removeEventListener('mouseup', this.moveEnd)
			document.removeEventListener('touchend', this.moveEnd)
    }
  }
}
</script>

<style lang="scss" scoped>
.puzzle-verify {
  overflow: hidden;
  display: inline-block;
  border: 1px solid #bbb;
  border-radius: 20px;
  .action-box {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 4px;
    .descript {
      font-size: 14px;
      color: #333;
    }
    .actions {
      display: flex;
      img {
        width: 18px;
        height: 18px;
        cursor: pointer;
      }
      .close-btn {
        margin-left: 10px;
      }
    }
  }
  .puzzle-box {
    overflow: hidden;
    position: relative;
    img {
      width: 100%;
      height: 100%;
    }
    #puzzle-box {
      position: absolute;
      top: 0;
      left: 0;
    }
    &-lost {
      position: absolute;
      top: 0;
      left: 0;
      canvas {
        position: absolute;
        top: 0;
        left: 0;
      }
    }
    .tips {
      position: absolute;
      left: 0;
      right: 0;
      bottom: -25px;
      padding: 0 10px;
      height: 25px;
      line-height: 25px;
      font-size: 12px;
      background: #e0e0e0;
      transition: all 0.4s;
      &.show-tips {
        bottom: 0;
      }
      p {
        margin: 0;
        &.success {
          color: #67C23A;
        }
        &.fail {
          color: #F56C6C;
        }
      }
    }
  }
  .slider-box {
    position: relative;
    padding: 15px 0;
    .slider-bar {
      height: 10px;
      border: 1px solid #c3c3c3;
      background: #e4e4e4;
      box-shadow: 0 1px 1px rgba(12, 10, 10, 0.2) inset;
      border-radius: 5px;
    }
    .slider-button {
      position: absolute;
      top: 50%;
      left: 5px;
      transform: translateY(-50%);
      width: 45px;
      height: 25px;
      background: #0095ff;
      border-radius: 20px;
      cursor: move;
      span {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
        height: 10px;
        width: 1px;
        background: #fff;
        &::before,
        &::after {
          content: '';
          position: absolute;
          left: -5px;
          display: block;
          height: 10px;
          width: 1px;
          background: #fff;
        }
        &::after {
          left: 5px;
        }
      }
    }
  }
}
</style>
