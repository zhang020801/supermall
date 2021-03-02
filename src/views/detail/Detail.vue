<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"></detail-nav-bar>
    <scroll class="content"
            ref="scroll"
            @scroll="contentScroll"
            :probe-type="3">
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods"></detail-base-info>
      <detail-shop-info :shop="shop"></detail-shop-info>
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"></detail-goods-info>
      <detail-param-info :param-info="paramInfo" ref="param"></detail-param-info>
      <detail-comment-info :comment-info="commentInfo" ref="comment"></detail-comment-info>
      <goods-list :goods="recommends" ref="recommend"></goods-list>
    </scroll>
    <detail-bottom-bar @addToCart="addToCart"></detail-bottom-bar>
    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>

  </div>
</template>

<script>
  import DetailNavBar from "./childComps/DetailNavBar";
  import {getDetail,Goods,Shop,GoodsParam,getRecommend} from "../../network/datail";
  import DetailSwiper from "./childComps/DetailSwiper";
  import DetailBaseInfo from "./childComps/DetailBaseInfo";
  import DetailShopInfo from "./childComps/DetailShopInfo";
  import Scroll from "components/common/scroll/Scroll";
  import DetailGoodsInfo from "./childComps/DetailGoodsInfo";
  import DetailParamInfo from "./childComps/DetailParamInfo";
  import DetailCommentInfo from "./childComps/DetailCommentInfo";
  import GoodsList from "components/content/goods/GoodsList";
  import DetailBottomBar from "./childComps/DetailBottomBar";
  import BackTop from "components/content/backtop/BackTop";
  export default {
    name: "Detail",
    components:{
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      Scroll,
      DetailGoodsInfo,
      DetailParamInfo,
      DetailCommentInfo,
      GoodsList,
      DetailBottomBar,
      BackTop

    },
    data(){
      return{
        iid: null,
        topImages:[],
        goods:{},
        shop:{},
        detailInfo:{},
        paramInfo:{},
        commentInfo:{},
        recommends: [],
        themeTopYs: [],
        currentIndex: 0,
        isShowBackTop: false,


      }
    },
    methods:{
      imageLoad(){
        this.$refs.scroll.refresh()
        this.themeTopYs.push(0);
        this.themeTopYs.push(this.$refs.param.$el.offsetTop)
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
      },
      titleClick(index){
        this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 500);
      },
      contentScroll(position){
        // console.log(position);
        const positionY = -position.y
        let  length = this.themeTopYs.length
        for(let i = 0;i < length; i++){
          if(this.currentIndex !== i && ((i < length - 1 && positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1])
            ||(i === length - 1 && positionY >= this.themeTopYs[i]))){
            this.currentIndex = i;
            this.$refs.nav.currentIndex = this.currentIndex
          }
        }

        this.isShowBackTop = (-position.y) > 1000
      },
      backClick(){
        // console.log(this.$refs.scroll)
        this.$refs.scroll.scrollTo(0,0,500)
      },
      addToCart(){
        // 1 获取购物车商品信息
        const product = {};
        product.image = this.topImages[0];
        product.title = this.detailInfo.title;
        product.desc = this.detailInfo.desc;
        product.price = this.detailInfo.realPrice;
        product.iid = this.iid;
        // 2 将商品添加到购物车
        this.$store.commit('addCart',product)
        console.log(this.$store.state)

      }
    },
    created() {
      //1 保存传入的iid
      this.iid = this.$route.params.iid

      //2 根据iid请求详情数据
      getDetail(this.iid).then(res => {
        // console.log(res);
        const data = res.result
        // 获取顶部图片轮播
        this.topImages = data.itemInfo.topImages

        //获取商品信息
        this.goods = new Goods(data.itemInfo, data.columns,data.shopInfo.services)

        //获取店铺信息
        this.shop = new Shop(data.shopInfo)

        //获取商品详情
        this.detailInfo = data.detailInfo

        //获取参数信息
        this.paramInfo = new GoodsParam(data.itemParams.info,data.itemParams.rule)

        //获取评论信息
        if (data.rate.cRate !== 0){
          this.commentInfo = data.rate.list[0]
        }
      })

      //3 请求推荐数据
      getRecommend().then(res => {
        // console.log(res);
        this.recommends = res.data.list
      })
    },
    mounted() {

    },
  }
</script>

<style scoped>
  #detail{
    position: relative;
    z-index: 9;
    background-color: #fff;
    height: 100vh;
  }
  .content{
    height: calc(100% - 44px - 58px);
    overflow: hidden;
  }
  .detail-nav{
    position: relative;
    z-index: 9;
    background-color: #fff;
  }
</style>
