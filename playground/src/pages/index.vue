<script setup lang="ts">
import { Loading, Upload } from '@element-plus/icons-vue'
import { fileToBlob } from 'lazy-js-utils'
import html2Canvas from 'html2canvas'
import { savePdf } from '../../../utils'

const fileRef = ref()
const loading = ref(false)
const newbase = ref<string>()
let canvasWrapper: HTMLCanvasElement
onMounted(() => {
  fileRef.value!.addEventListener('change', update)
})

async function update() {
  const files = fileRef.value.files
  const imgBlobs = await Promise.all([...files].map(file => fileToBlob(file)))
  loading.value = true

  let count = imgBlobs.length

  const wrapper = document.querySelector('.wrapper')!

  for (const imgBlob of imgBlobs) {
    const uri = URL.createObjectURL(imgBlob)
    const image = new Image()
    image.src = uri
    image.style.width = 'fit-content'

    wrapper.appendChild(image)
    image.onerror = () => {
      count--
      if (count === 0)
        loading.value = false
    }
  }

  await html2Canvas(wrapper as any, {
    scale: 2,
    allowTaint: true,
    useCORS: true,
  }).then((canvas) => {
    console.log('canvas-----------', canvas)
    // wrapper?.appendChild(canvas)
    canvasWrapper = canvas
    const contentWidth = canvas.width
    const contentHeight = canvas.height
    const pageHeight = contentWidth / 592.28 * 841.89
    console.log('contentWidth', contentWidth, 'contentHeight', contentHeight, 'pageHeight', pageHeight)
  })
  loading.value = false
}
const upload = () => {
  document.getElementById('file')?.click()
}

const download = () => {
  savePdf(canvasWrapper, 'title')
}
</script>

<template>
  <div class="nihao">
    <div v-show="loading" class="loading">
      <el-icon class="is-loading" size="28px">
        <Loading />
      </el-icon>
    </div>
    <el-button v-if="!newbase" size="large" @click="upload">
      Upload<el-icon class="el-icon--right">
        <Upload />
      </el-icon>
    </el-button>
    <el-button v-if="!newbase" size="large" @click="download">
      download
    </el-button>
    <input id="file" ref="fileRef" type="file" accept="image/*" multiple hidden>
    <div p10 w-full min-h-200>
      <div flex flex-col items-center w-full h-full gap5 py10 border class="wrapper border-[#333]" />
    </div>
  </div>
</template>

<style scoped>
.loading {
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10;
}
</style>
