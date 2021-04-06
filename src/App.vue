<template>
  <div id="app">
    <div class="set-wrap">
      <label for="bg">
        背景：
        <input type="file" accept=".jpg,.jpeg,.png" name="bg" id="bg" @change="onBgFileChange">
      </label>
      <label for="image" v-if="bgUrl">
        图片：
        <input type="file" accept=".jpg,.jpeg,.png" name="image" id="image" @change="onImageFileChange">
      </label>
      <label for="main-title">
        标题文案：
        <input type="text" id="main-title" v-model="mainTitle">
      </label>
      <label for="sub-title-0">
        副标题文案：
        <input type="text" id="sub-title-0" v-model="subTitle">
      </label>
    </div>
    <div class="design-wrap">
      <div class="poster-wrap" :style="posterStyle">
        <VueDraggableResizable
          v-show="mainTitle"
          w="auto"
          h="auto"
          :resizable="false"
        >
          <div
            contenteditable="true"
            class="text-input"
            ref="drop-main-title"
            @blur="onMainTitleInputBlur"
          ></div>
        </VueDraggableResizable>
        <VueDraggableResizable
          v-show="subTitle"
          w="auto"
          h="auto"
          :resizable="false"
        >
          <div
            contenteditable="true"
            class="text-input"
            ref="drop-sub-title"
            @blur="onSubTitleInputBlur"
          ></div>
        </VueDraggableResizable>
        <VueDraggableResizable v-for="(item, index) in images" :w="item.width" :h="item.height" :key="index" @resizing="onImageResize(index, ...arguments)">
          <div class="draw-image" :style="item.imageStyle"/>
        </VueDraggableResizable>
      </div>
    </div>
  </div>
</template>

<script>
import VueDraggableResizable from 'vue-draggable-resizable';
import 'vue-draggable-resizable/dist/VueDraggableResizable.css'

export default {
  name: 'App',
  components: {
    VueDraggableResizable
  },
  data () {
    return {
      posterStyle: {},
      mainTitle: '',
      subTitle: '',
      bgUrl: null,
      images: []
    }
  },
  computed: {
    mainTitleDom () {
      return this.$refs['drop-main-title']
    },
    subTitleDom () {
      return this.$refs['drop-sub-title']
    },
  },
  watch: {
    mainTitle () {
      this.mainTitleDom.innerText = this.mainTitle
    },
    subTitle () {
      this.subTitleDom.innerText = this.subTitle
    }
  },
  methods: {
    onMainTitleInputBlur () {
      this.mainTitle = this.mainTitleDom.innerText
    },
    onSubTitleInputBlur () {
      this.subTitle = this.subTitleDom.innerText
    },
    onImageResize (index, left, top, width, height) {
      this.images[index].width = width
      this.images[index].height = height
    },
    async onBgFileChange (e) {
      this.images = []
      this.mainTitle = ''
      this.subTitle = ''
      this.bgUrl && window.URL.revokeObjectURL(this.bgUrl)
      const fileList = e.target.files;
      const file = fileList[0];
      this.bgUrl = URL.createObjectURL(file)
      const imageRect = await this.getFileImageRect(this.bgUrl)
      const bgRect = this.getBgRect(imageRect)
      this.posterStyle = {
        width: bgRect.w + 'px',
        height: bgRect.h + 'px',
        backgroundImage: `url(${this.bgUrl})`
      }
      e.target.value = ''
    },
    async onImageFileChange (e) {
      const fileList = e.target.files;
      const file = fileList[0];
      const imageUrl = URL.createObjectURL(file)
      const imageRect = await this.getFileImageRect(imageUrl)
      const bgRect = this.getBgRect(imageRect, 100, 200)
      const imageStyle = {
        backgroundImage: `url(${imageUrl})`
      }
      this.images.push({
        width: bgRect.w,
        height: bgRect.h,
        imageStyle
      })
      e.target.value = ''
    },
    getFileImageRect (src) {
      return new Promise((resolve) => {
        const image = new Image()
        image.onload = () => {
          const width = image.width
          const height = image.height
          resolve({
            width,
            height
          })
        }
        image.src = src
      })
    },
    getBgRect (imageRect, maxWidth = 400, maxHeight = 800) {
      const { width, height } = imageRect
      let w = width
      let h = height
      if (width > height) {
        if (width > maxWidth) {
          h = h / w * maxWidth
          w = maxWidth
        }
      } else {
        if (height > maxHeight) {
          w = w / h * maxHeight
          h = maxHeight
        }
      }
      return {
        w,
        h
      }
    }
  }
}
</script>

<style>
#app {
  text-align: center;
  margin-top: 60px;
}
.set-wrap .upload-btn {
  margin-bottom: 20px;
}
.design-wrap {
  display: inline-block;
  margin-top: 20px;
}
.poster-wrap {
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  overflow: hidden;
}
.text-input {
  /* border: 1px solid rgba(235,238,245,0.60); */
}
.draw-image {
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  overflow: hidden;
}
.no-style {
  width: auto;
  height: auto;
  padding: 0;
  margin: 0;
  outline: none;
  border: none;
  background: none;
}
</style>
