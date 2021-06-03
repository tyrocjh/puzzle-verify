<template>
  <div id="app">
    <div class="puzzle-box">
      <p>默认用法：</p>
      <PuzzleVerify />
    </div>

    <div class="puzzle-box">
      <p>多个图片&回调：</p>
      <PuzzleVerify :img-list="imgList" 
        @handle-success="handleSuccess"
        @handle-error="handleError" />
    </div>

    <div class="puzzle-box">
      <p>拼图形状：</p>
      <PuzzleVerify :img-list="imgList" puzzle-type="puzzle" />
    </div>

    <div class="puzzle-box">
      <p>误差范围：</p>
      <PuzzleVerify :img-list="imgList" :deviation="20" />
    </div>

    <div class="puzzle-box">
      <p>各属性大小：</p>
      <PuzzleVerify :img-list="imgList" :img-width="200" :img-height="100" 
        :box-padding="10" :puzzle-size="30" :puzzle-radius="8" :default-offset-left="5" />
    </div>
  </div>
</template>

<script>
import PuzzleVerify from './components/PuzzleVerify.vue'

export default {
  name: 'App',
  components: {
    PuzzleVerify
  },
  data() {
    return {
      imgList: [
        require('@/assets/cat.jpg'),
        require('@/assets/cat2.jpg'),
        require('@/assets/onepiece.jpg')
      ]
    }
  },
  methods: {
    handleSuccess() {
      console.info('success')
    },
    handleError() {
      console.info('error')
    }
  }
}
</script>

<style scoped>
  .puzzle-box {
    display: inline-block;
    margin: 10px;
  }
</style>
