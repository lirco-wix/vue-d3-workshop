<template>
  <BasePage>
    <div slot="readme" ref="readme" v-html="readme"></div>
    <div slot="example" ref="example"><WordCloud v-bind="cloudOptions" /></div>
  </BasePage>
</template>

<script>
import readme from './readme.md'
import BasePage from '@/views/BasePage.vue'
import WordCloud from '@/components/d3/finished/WordCloud.vue'
export default {
  data() {
    return {
      readme,
      cloudOptions: {
        value:     null,
        size:      [1000, 1000],
        rotate:    () => (Math.random() > 0.5 ? 0 : (90 * Math.PI) / 180),
        immediate: true
      }
    }
  },
  components: {
    WordCloud,
    BasePage
  },
  mounted() {
    this.$nextTick(() => {
      const rect = this.$refs.example.getBoundingClientRect()
      this.cloudOptions.size = [rect.width, rect.height]
      this.cloudOptions.value = this.$refs.readme.innerText
    })
  }
}
</script>
