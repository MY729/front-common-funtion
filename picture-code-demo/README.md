# 使用canvas一步步实现图片打码功能

## 准备工作

demo 基于 vue + elelment-ui

首先创建一个html文件, 并引入 vue 和 elelment-ui(注意还有样式文件)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <!-- elelment-ui样式 -->
  <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
</head>
<body>
  
</body>
<!-- 引入vue -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
<!-- 引入element-ui -->
<script src="https://unpkg.com/element-ui/lib/index.js"></script>
</html>
```

接下来就可以写我们的打码功能啦

## 实现思路

1. 创建canvas画布，并将要打码的图片绘制上去
2. 监听鼠标在图片上的点击，移动、松开事件，在canvas画布上绘制要打码的区域
3. 处理绘制的打码区域
4. 保存打码后的图片

## 将要打码的图片绘制到canvas画布上

```js
// 初始化 绘制图片
toCode (currentImg) {
  this.$nextTick(() => {
    // 获取将要绘制的canvas的父元素节点
    let parentId = document.getElementById('parentId')
    // 初始化图片
    let drawImg = new Image()
    drawImg.setAttribute('crossOrigin', 'anonymous')
    drawImg.crossOrigin = 'Anonymous'
    drawImg.src = currentImg
    // 创建canvas元素并添加到父节点中
    let addCanvas = document.createElement('canvas')
    parentId.appendChild(addCanvas)
    let canvas = parentId.lastElementChild
    canvas.id = 'imgCanvas'
    if (canvas.getContext) {
      let ctx = canvas.getContext('2d')
      // 绘制图片
      drawImg.onload = function () {
        canvas.width = 720
        canvas.height = 500
        ctx.drawImage(drawImg, 0, 0, 720, 500)
      }
    }
  })
}
```

## 点击打码按钮，绘制打码区域

思路：

  1. 鼠标点击，获取点击时的坐标，每次点击前可能会存在打过码的区域，先清除画布，重新绘制图片
  2. 鼠标移动，开始绘制打码的矩形，通过移动的坐标和上面点击的点坐标确定绘制的矩形坐标和宽高
  3. 将绘制的打码矩形，分割成一个个宽高15像素的小正方形，并给每个小正方形生产随机颜色
  4. 鼠标松开，停止绘制矩形

```js
// 打码
dialogCode (img) {
  let parentId = document.getElementById('parentId')
  let canvas = document.getElementById('imgCanvas')
  if (canvas.getContext) {
    let ctx = canvas.getContext('2d')
    let drawImage = new Image()
    drawImage.crossOrigin = 'Anonymous'
    drawImage.src = img
    drawImage.onload = () => {
      ctx.drawImage(drawImage, 0, 0, 720, 500)
    }
    // 鼠标点击
    parentId.onmousedown = e => {
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      ctx.drawImage(drawImage, 0, 0, 720, 500)
      this.flag = true
      this.clickX = e.offsetX // 鼠标点击时的X
      this.clickY = e.offsetY // 鼠标点击时的Y
    }
    // 鼠标松开
    parentId.onmouseup = () => {
      this.flag = false
    }
    // 鼠标按下
    parentId.onmousemove = e => {
      if (this.flag) {
        ctx.clearRect(0, 0, canvas.width, canvas.height)
        ctx.drawImage(drawImage, 0, 0, 720, 500)
        ctx.beginPath()
        let pixels = [] // 二维数组，每个子数组有5个值（绘制矩形左上角的X坐标、y坐标，矩形的宽、高，生成的4位随机数用于颜色值）
        for (let x = 0; x < (e.offsetX - this.clickX) / 15; x++) {
          for (let y = 0; y < (e.offsetY - this.clickY) / 15; y++) {
            pixels.push([(x * 15 + this.clickX), (y * 15 + this.clickY), 15, 15, Math.floor(Math.random() * 9999)])
          }
          for (let y = 0; y > (e.offsetY - this.clickY) / 15; y--) {
            pixels.push([(x * 15 + this.clickX), (y * 15 + this.clickY), 15, 15, Math.floor(Math.random() * 9999)])
          }
        }
        for (let x = 0; x > (e.offsetX - this.clickX) / 15; x--) {
          for (let y = 0; y > (e.offsetY - this.clickY) / 15; y--) {
            pixels.push([(x * 15 + this.clickX), (y * 15 + this.clickY), 15, 15, Math.floor(Math.random() * 9999)])
          }
          for (let y = 0; y < (e.offsetY - this.clickY) / 15; y++) {
            pixels.push([(x * 15 + this.clickX), (y * 15 + this.clickY), 15, 15, Math.floor(Math.random() * 9999)])
          }
        }
        // 遍历数组绘制小正方形块
        for (let i = 0; i < pixels.length; i++) {
          ctx.fillStyle = '#bf' + pixels[i][4]
          ctx.fillRect(pixels[i][0], pixels[i][1], pixels[i][2], pixels[i][3])
        }
        ctx.fill()
        ctx.closePath()
      }
    }
  }
}
```

## 保存

```js
// 保存
dialogUpload () {
  let canvas = document.getElementById('imgCanvas')
  let tempImg = canvas.toDataURL('image/png')
  let imgURL = document.getElementById('imgURL')
  imgURL.crossOrigin = 'Anonymous'
  imgURL.src = tempImg
}
```

## 源码

复制到html文件可预览

```html

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>使用canvas一步步实现图片打码功能</title>
  <!-- elelment-ui样式 -->
  <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
  <style type="text/css">
    .rc-code__buttons {
      margin: 20px;
    }
  </style>
</head>
<body>
  <div id="app">
    <div class="rc-code__buttons">
      <h1>vue项目中使用canvas一步步实现图片打码功能</h1>
      <el-button type="primary" @click="dialogCode(data.img_url)">打码</el-button>
      <el-button type="success" @click="dialogUpload()">保存</el-button>
    </div>
    <el-row>
      <el-col :span="12"><h3>点击打码按钮，在图片上绘制打码区域； 点击保存，生成打码后的图片</h3></el-col>
      <el-col :span="12"><h3>保存后的图片</h3></el-col>
      <el-col :span="12"><div id="parentId"></div></el-col>
      <el-col :span="12"><img id="imgURL"/></el-col>
    </el-row>
  </div>
</body>
<!-- 引入vue -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
<!-- 引入element-ui -->
<script src="https://unpkg.com/element-ui/lib/index.js"></script>
<script>
new Vue({
  el: '#app',
  data () {
    return {
      data: {
        img_url: 'https://avatars0.githubusercontent.com/u/26196557?s=460&v=4'
      },
      flag: false, // 是否绘制矩形
      clickX: '', // 开始绘制矩形时，鼠标点击时的x坐标
      clickY: '' // 开始绘制矩形时，鼠标点击时的y坐标
    }
  },
  mounted() {
    this.toCode(this.data.img_url)
  },
  methods: {
    // 初始化 绘制图片
    toCode (currentImg) {
      this.$nextTick(() => {
        let parentId = document.getElementById('parentId')
        let drawImg = new Image()
        drawImg.setAttribute('crossOrigin', 'anonymous')
        drawImg.crossOrigin = 'Anonymous'
        drawImg.src = currentImg
        let addCanvas = document.createElement('canvas')
        parentId.appendChild(addCanvas)
        let canvas = parentId.lastElementChild
        canvas.id = 'imgCanvas'
        if (canvas.getContext) {
          let ctx = canvas.getContext('2d')
          drawImg.onload = function () {
            canvas.width = 720
            canvas.height = 500
            ctx.drawImage(drawImg, 0, 0, 720, 500)
          }
        }
      })
    },
    // 打码
    dialogCode (img) {
      let parentId = document.getElementById('parentId')
      let canvas = document.getElementById('imgCanvas')
      if (canvas.getContext) {
        let ctx = canvas.getContext('2d')
        let drawImage = new Image()
        drawImage.crossOrigin = 'Anonymous'
        drawImage.src = img
        drawImage.onload = () => {
          ctx.drawImage(drawImage, 0, 0, 720, 500)
        }
        parentId.onmousedown = e => {
          ctx.clearRect(0, 0, canvas.width, canvas.height)
          ctx.drawImage(drawImage, 0, 0, 720, 500)
          this.flag = true
          this.clickX = e.offsetX // 鼠标点击时的X
          this.clickY = e.offsetY // 鼠标点击时的Y
        }
        parentId.onmouseup = () => {
          this.flag = false
        }
        parentId.onmousemove = e => {
          if (this.flag) {
            ctx.clearRect(0, 0, canvas.width, canvas.height)
            ctx.drawImage(drawImage, 0, 0, 720, 500)
            ctx.beginPath()
            let pixels = [] // 二维数组，每个子数组有5个值（绘制矩形左上角的X坐标、y坐标，矩形的宽、高，生成的4位随机数用于颜色值）
            for (let x = 0; x < (e.offsetX - this.clickX) / 15; x++) {
              for (let y = 0; y < (e.offsetY - this.clickY) / 15; y++) {
                pixels.push([(x * 15 + this.clickX), (y * 15 + this.clickY), 15, 15, Math.floor(Math.random() * 9999)])
              }
              for (let y = 0; y > (e.offsetY - this.clickY) / 15; y--) {
                pixels.push([(x * 15 + this.clickX), (y * 15 + this.clickY), 15, 15, Math.floor(Math.random() * 9999)])
              }
            }
            for (let x = 0; x > (e.offsetX - this.clickX) / 15; x--) {
              for (let y = 0; y > (e.offsetY - this.clickY) / 15; y--) {
                pixels.push([(x * 15 + this.clickX), (y * 15 + this.clickY), 15, 15, Math.floor(Math.random() * 9999)])
              }
              for (let y = 0; y < (e.offsetY - this.clickY) / 15; y++) {
                pixels.push([(x * 15 + this.clickX), (y * 15 + this.clickY), 15, 15, Math.floor(Math.random() * 9999)])
              }
            }
            for (let i = 0; i < pixels.length; i++) {
              ctx.fillStyle = '#bf' + pixels[i][4]
              ctx.fillRect(pixels[i][0], pixels[i][1], pixels[i][2], pixels[i][3])
            }
            ctx.fill()
            ctx.closePath()
          }
        }
      }
    },
    // 保存
    dialogUpload () {
      let canvas = document.getElementById('imgCanvas')
      let tempImg = canvas.toDataURL('image/png')
      let imgURL = document.getElementById('imgURL')
      imgURL.crossOrigin = 'Anonymous'
      imgURL.src = tempImg
    }
  }
})
</script>
</html>
```
