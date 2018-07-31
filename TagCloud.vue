/*
 * @Author: wanghaoyu 
 * @Date: 2018-07-31 10:19:10 
 * @Last Modified by: wanghaoyu
 * @Last Modified time: 2018-07-31 15:55:26
 */

 /*
 @params: （* 表示必填）
  *size: 整个标签云的尺寸（正方形）
  *tagsNum: 渲染标签的个数
  *tagsContent: 标签的数据，包括名字和跳转地址
    传入格式要求：（text：标签名，href：跳转路径）
      [{ text: '测试', href: 'https://github.com/Howie126313' }]
  color: 标签文字颜色   
  hoverColor: 标签移入后文字颜色（默认 #4c84ff） 
 @example:
  <TagCloud 
  size='200'
  tagsNum='20'
  :tagsContent="tags" />
 */
<template>
  <div id="tagCloud">
    <div class="cloudWrap">
      <svg 
      :width="svgWidth" 
      :height="svgHeight"
      @mousemove="mousemove($event)">
        <a 
        v-for="(tag, index) in tags" 
        :key="index" 
        :href="tag.href"
        target="_blank"
        @mouseover="tagOver(tag)"
        @mouseout="tagOut(tag)">
          <text
          :x="tag.x"
          :y="tag.y"
          :font-size='14 || 20 * (RADIUS/(2*RADIUS-tag.z))'
          :fill-opacity='((400+tag.z)/600)'>
            <tspan v-if="tag.hover" :fill="hoverColor">{{tag.text}}</tspan>          
            <tspan v-else :fill="textColor">{{tag.text}}</tspan>
          </text>
        </a>
      </svg>
    </div>
  </div>
</template>

<script>

export default {
  props: {
    size: {
      required: true,
      type: String
    },
    tagsContent: {
      type: Array,
      required: true
    },
    tagsNum: {
      type: String,
      required: true
    },
    color: {
      type: String
    },
    hoverColor: {
      type: String,
      default: '#4c84ff'
    }
  },
  data() {
    return {
      svgWidth: '',
      svgHeight: '',
      RADIUS: '',
      speedX: Math.PI / 360,
      speedY: Math.PI / 360,
      tags: [],
      timeout: '',
      textColor: ''
    }
  },
  computed: {
    centerX() {
      return this.svgWidth / 2 - 15
    },
    centerY() {
      return this.svgHeight / 2
    }
  },
  created() {
    this.svgWidth = this.size
    this.svgHeight = this.size
    this.RADIUS = this.size * 0.4
    var tags = []
    for (let i = 0; i < this.tagsNum; i++) {
      const tag = {}
      const k = -1 + (2 * (i + 1) - 1) / this.tagsNum
      const a = Math.acos(k)
      const b = a * Math.sqrt(this.tagsNum * Math.PI)
      tag.x = this.centerX + this.RADIUS * Math.sin(a) * Math.cos(b)
      tag.y = this.centerY + this.RADIUS * Math.sin(a) * Math.sin(b)
      tag.z = this.RADIUS * Math.cos(a)
      tag.hover = false
      if (this.tagsContent && this.tagsContent[i].text && this.tagsContent[i].href) {
        tag.text = this.tagsContent[i].text
        tag.href = this.tagsContent[i].href
        tags.push(tag)
      } else {
        console.warn('tagsContent 出错')
        return
      }
    }
    this.tags = tags
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      this.color ? this.textColor = this.color : ''
      this.timeout = setInterval(() => {
        this.rotateX(this.speedX)
        this.rotateY(this.speedY)
      }, 50)
    },
    rotateX(x) {
      const cos = Math.cos(x)
      const sin = Math.sin(x)
      for (const tag of this.tags) {
        tag.y = (tag.y - this.centerY) * cos - tag.z * sin + this.centerY
        tag.z = tag.z * cos + (tag.y - this.centerY) * sin
      }
    },
    rotateY(y) {
      const cos = Math.cos(y)
      const sin = Math.sin(y)
      for (const tag of this.tags) {
        tag.x = (tag.x - this.centerX) * cos - tag.z * sin + this.centerX
        tag.z = tag.z * cos + (tag.x - this.centerX) * sin
      }
    },
    mousemove(e) {
      const x = e.clientX - this.centerX
      const y = e.clientY - this.centerY
      this.speedX = x * 0.0001 > 0 ? Math.min(this.RADIUS * 0.00002, x * 0.0001) : Math.max(-this.RADIUS * 0.00002, x * 0.0001)
      this.speedY = y * 0.0001 > 0 ? Math.min(this.RADIUS * 0.00002, y * 0.0001) : Math.max(-this.RADIUS * 0.00002, y * 0.0001)
    },
    tagOver(tag) {
      clearInterval(this.timeout)
      tag.hover = true
    },
    tagOut(tag) {
      tag.hover = false
      this.timeout = setInterval(() => {
        this.rotateX(this.speedX)
        this.rotateY(this.speedY)
      }, 17)
    }
  }
}
</script>
