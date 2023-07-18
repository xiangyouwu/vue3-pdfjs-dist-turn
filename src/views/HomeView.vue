<template>
  <div class="box">
    <div id="flipbook"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick, onMounted } from 'vue'
import $ from 'jquery'
// eslint-disable-next-line @typescript-eslint/no-unused-vars
import '@/utils/turnjs4/lib/turn.js'
import * as pdfjs from 'pdfjs-dist'
import url_01 from '@/assets/pdf/03-1.pdf'

pdfjs.GlobalWorkerOptions.workerSrc = '/pdf.worker.js' //这个文件地址一定要找对，我是放在public里面所以用 /

const pdfInit = async (url) => {
  // 获取元素
  const pdfContainer = document.querySelector('#flipbook')
  if (!pdfContainer) {
    return
  }

  const loadingTask = pdfjs.getDocument({
    url: url,
  })
  const pdf = await loadingTask.promise
  const container: any = document.querySelector('#flipbook')

  for (let index = 0; index < pdf.numPages; index++) {
    const page = await pdf.getPage(index + 1)
    const viewport = page.getViewport({ scale: 0.8 })
    console.log(viewport)

    const divPage = document.createElement('div')
    divPage.classList.add('page')
    const divPageContent = document.createElement('div')
    divPageContent.classList.add('page-content')
    const canvas = document.createElement('canvas')
    canvas.width = viewport.width
    canvas.height = viewport.height
    const context = canvas.getContext('2d')
    if (!context) {
      return
    }
    const renderContext = {
      canvasContext: context,
      viewport: viewport,
    }
    await page.render(renderContext).promise
    divPageContent.appendChild(canvas)
    divPage.appendChild(divPageContent)
    container.appendChild(divPage)
  }
}

// 页数
const pageCav = ref<any>([])

const currentPage = ref(1)

onMounted(async () => {
  await pdfInit(url_01)

  pageCav.value = document.querySelectorAll('#flipbook .page')
  await onTurn()
})

const onTurn = () => {
  $('#flipbook').turn({
    autoCenter: true, //自动居中, 默认false
    height: 673, //高度
    width: 952, //宽度
    display: 'double', //单页显示/双页显示  single/double
    elevation: 50,
    duration: 500, //翻页速度(毫秒), 默认600ms
    gradients: true, //翻页时的阴影渐变, 默认true
    acceleration: true, //硬件加速, 默认true, 如果是触摸设备设置为true
    page: 1, //设置当前显示第几页
    pages: pageCav.value.length, //总页数
    turnCorners: 'bl,br,tl,tr,l,r', // 设置可翻页的页角(都试过了，乱写 4个角都能出发卷起)， bl,br   tl,tr   bl,tr
    when: {
      //监听事件
      turning: async function (e, page, view) {
        console.log(e, page, view)
        // 翻页前触发
      },
      turned: function (e, page) {
        console.log(e, page, pageCav.value.length)
        currentPage.value = page
        // 翻页后触发
      },
    },
  })
}
</script>

<style scoped lang="less">
.box {
  width: 952px;
  box-shadow: 0 4px 10px #666;
  margin: 100px auto;
  overflow: hidden;
}
</style>
