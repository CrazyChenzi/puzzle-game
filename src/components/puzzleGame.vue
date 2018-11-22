<template>
  <div>
    <div>
      <label>行：</label><input type="number" class="header-container" v-model="cellRow">
      <label>列：</label><input type="number" class="header-container" v-model="cellCol">
      <button @click="play" class="header-container">开始</button>
    </div>
    <div id="play_area" :style="{width: areaWidth + 'px', height: areaHeight + 'px'}"></div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      playArea: null,
      cellArr: [],  // 保存html元素
      cellRow: 3, // 行
      cellCol: 3, // 列
      img: '',  // 图片地址
      imgArr: [],  // 用于随机图片顺序
      cellWidth: 0, // 每一个图的宽度
      cellHeight: 0,
      cellIndex: null, // 当前拖动元素的index
      nextIndex: null,  // 元素的最终的index 用于交换位置
      cellLeft: '', // 元素的left坐标值
      cellTop: '', 
      ranArr: [], // 用于存放元素的数组
      areaWidth: 300, // play_area的宽度
      areaHeight: 300 // play_area的高度
    }
  },
  methods: {
    // 初始化方法
    init () {
      let dom
      this.cellWidth = this.areaWidth / this.cellCol
      this.cellHeight = this.areaHeight / this.cellRow
      for (let i = 0; i < this.cellRow; i++) {
        for (let j = 0; j < this.cellCol; j++) {
          this.imgArr.push(i * this.cellCol + j)
          dom = document.createElement('div')
          dom.className = 'play_cell'
          dom.style.width = this.cellWidth + 'px'
          dom.style.height = this.cellHeight + 'px'
          dom.style.left = j * this.cellWidth + 'px'
          dom.style.top = i * this.cellHeight + 'px'
          dom.style.background = 'url(' + this.img + ')'
          dom.style.backgroundPosition = (-j) * this.cellWidth + 'px ' + (-i) * this.cellHeight + 'px'
          dom.ondragstart = function () {
            return false
          }
          this.cellArr.push(dom)
        }
      }
      this.playArea.append(...this.cellArr)
    },
    play () {
      this.cellArr.forEach((child) => {
        this.playArea.removeChild(child)
      })
      this.imgArr = []
      this.cellArr = []
      this.init()
      this.randomImg()
      this.bindCell()
    },
    // 随机打乱元素顺序
    randomImg () {
      let arr, random
      arr = this.imgArr.slice()
      this.ranArr = []
      for (let i = 0, ilen = arr.length; i < ilen; i++) {
        random = Math.floor(Math.random() * arr.length)
        this.ranArr.push(arr[random])
        let x = (-arr[random] % this.cellCol) * this.cellWidth + 'px'
        let y = (-Math.floor(arr[random] / this.cellCol)) * this.cellHeight + 'px'
        this.cellArr[i].style.backgroundPosition = x + ' ' +  y
        arr.splice(random, 1)
      }
    },
    // 绑定拖动方法
    bindCell () {
      const _this = this
      for (let i = 0, len = this.cellArr.length; i < len; i++) {
        this.cellArr[i].onmouseover = function () {
          _this.cellArr[i].style.filter = 'alpha(opacity=80)'
          _this.cellArr[i].style.opacity = '.8'
          _this.cellArr[i].style.boxShadow = '0px 0px 8px #000'
          _this.cellArr[i].style.zIndex = '90'
        }
        this.cellArr[i].onmouseout = function () {
          _this.cellArr[i].style.filter = ''
          _this.cellArr[i].style.opacity = ''
          _this.cellArr[i].style.boxShadow = ''
          _this.cellArr[i].style.zIndex = '80'
        }
        this.cellArr[i].onmousedown = function (e) {
          _this.cellDown(e, _this.cellArr[i])
          e.preventDefault() 
          return false
        }
      }
    },
    // 拖动
    cellDown (e, _cell) {
      const _this = this
      const parentLeft = this.playArea.offsetLeft
      const parentTop = this.playArea.offsetTop
      let _x = e.pageX - this.playArea.offsetLeft - _cell.offsetLeft
      let _y = e.pageY - this.playArea.offsetTop - _cell.offsetTop
      this.cellLeft = _cell.style.left.split('px')[0]
      this.cellTop = _cell.style.top.split('px')[0]
      this.cellIndex = Math.floor(parseInt(this.cellTop) / this.cellHeight) * this.cellCol + Math.floor(parseInt(this.cellLeft) / this.cellWidth)
      _cell.style.zIndex = '99'
      document.onmousemove = function (e) {
        _cell.style.left = e.pageX - parentLeft - _x + 'px'
        _cell.style.top = e.pageY - parentTop - _y + 'px'
      }
      document.onmouseup = function (e) {
        document.onmousemove = null
        document.onmouseup = null
        if(e.pageX - parentLeft < 0 || e.pageX - parentLeft > _this.areaWidth || e.pageY - parentTop < 0 || e.pageY - parentTop > _this.areaHeight) {
          _this.returnCell(_cell)
          return
        }
        let _tx, _ty, _ti, _tj
        _tx = e.pageX - parentLeft
        _ty = e.pageY - parentTop

        _ti = Math.floor(_ty / _this.cellHeight)
        _tj = Math.floor(_tx / _this.cellWidth)

        _this.nextIndex = _ti * _this.cellCol + _tj
        if (_this.nextIndex === _this.cellIndex) {
          _this.returnCell(_cell)
        } else {
          _this.changeCell()
        }
      }
    },
    // 不需更换位置时返回到原位置
    returnCell (_cell) {
      const timer = setInterval(() => {
        _cell.style.left = this.cellLeft + 'px'
        _cell.style.top = this.cellTop + 'px'
        clearInterval(timer)
      })
    },
    // 交换位置
    changeCell () {
      let _tc = this.cellArr[this.cellIndex]
      let _tl = this.cellLeft
      let _tt = this.cellTop
      let _nc = this.cellArr[this.nextIndex]
      let _nl = (this.nextIndex % this.cellCol) * this.cellWidth
      let _nt = Math.floor(this.nextIndex / this.cellCol) * this.cellHeight
      _nc.style.zIndex = '98'

      this.ranArr[this.nextIndex] = this.ranArr[this.nextIndex] + this.ranArr[this.cellIndex]
      this.ranArr[this.cellIndex] = this.ranArr[this.nextIndex] - this.ranArr[this.cellIndex]
      this.ranArr[this.nextIndex] = this.ranArr[this.nextIndex] - this.ranArr[this.cellIndex]
      
      _tc.style.left = _nl + 'px'
      _tc.style.top = _nt + 'px'

      _nc.style.left = _tl + 'px'
      _nc.style.top = _tt + 'px'
      this.cellArr[this.nextIndex] = _tc
      this.cellArr[this.cellIndex] = _nc

      const timer = setInterval(() => {
        _nc.style.zIndex = ''
        _nc.style.filter = ''
        _nc.style.opacity = ''
        _nc.style.boxShadow = ''
        _tc.style.zIndex = ''
        _tc.style.filter = ''
        _tc.style.opacity = ''
        _tc.style.boxShadow = ''
        clearInterval(timer)
      })
      // 判断是否拼图成功
      if (this.ranArr.join() == this.imgArr.join()) {
        alert('success')
      }
      // 重置绑定的mouse方法
      this.bindCell()
    }
  },
  mounted () {
    this.playArea = this.$el.querySelector('#play_area')
    this.img = require('../assets/puzzleGame/puzzle.jpg')
    this.init()
  }
};
</script>
<style>
.header-container {
  margin-right: 10px;
  margin-bottom: 10px;
}
#play_area {
  position: relative;
  margin: auto;
  background: #fefefe;
  border-radius: 2px;
  color: black;
  border: 1px solid #fff;
  border: 1px solid #e5e5e5;
  cursor: default;
}
#play_area .play_cell{
  width:48px;
  height:48px;
  border:1px solid #fff;
  border-radius:4px;
  position:absolute;
  background-position: 5px 5px;
  cursor: default;
  z-index:80;
  transition-property:background-position;
  transition-duration:300ms;
  transition-timing-function:ease-in-out;
}
#play_area .play_cell.hover{
  filter: alpha(opacity=80);
  opacity:.8;
  box-shadow: 0px 0px 8px #000;
  z-index:90;
  *border:1px solid #09F;
}
</style>

