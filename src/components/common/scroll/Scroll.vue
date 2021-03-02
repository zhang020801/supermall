<template>
  <div>
    <div class="wrapper" ref="wrapper">
      <div class="content">
        <slot></slot>
      </div>
    </div>
  </div>
</template>

<script>
  import BScroll from 'better-scroll'
  export default {
    name: "Scroll",
    props:{
      probeType:{
        type: Number,
        default: 0
      },
      pullUpLoad:{
        type:Boolean,
        default: false
      }
    },
    data(){
      return{
        scroll: null
      }
    },
    mounted() {
      //1 创建BScroll对象
      if (!this.$refs.wrapper) return
      this.scroll = new BScroll(this.$refs.wrapper, {
        probeType: this.probeType,
        click: true,
        pullUpLoad: this.pullUpLoad
      })
      //2 监听滚动位置
      this.scroll.on('scroll',(position) => {
        // console.log(position);
        this.$emit('scroll',position)
      })
      //3 监听上拉事件
      this.scroll.on('pullingUp',() => {
        this.$emit('pullingUp')
      })

    },

    methods:{
      refresh(){
        this.scroll && this.scroll.refresh && this.scroll.refresh()
      },
      scrollTo(x, y, time=300){
        this.scroll && this.scroll.scrollTo(x * 1, y *1, time)
      },
      finishPullUp(){
        this.scroll && this.scroll.finishPullUp()
      },
      getScrollY(){
        return this.scroll ? this.scroll.y : 0
      }
    }
  }
</script>

<style scoped>
  .wrapper{
    height: 100%;
  }
</style>
