<template>
  <view>
    <view class="goods-list">
      <view v-for="(goods, i) in goodsList" :key="i" @click="gotoDetail(goods)">
        <my-goods :goods="goods"></my-goods>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        //请求参数
        queryobj: {
          query: '',
          cid: '',
          pagenum: 1,
          pagesize: 10
        },
        //商品列表的数据
        goodsList: [],
        //总数量，用来实现分页
        total: 0,
        // 默认的空图片
        defaultPic: 'https://img3.doubanio.com/f/movie/8dd0c794499fe925ae2ae89ee30cd225750457b4/pics/movie/celebrity-default-medium.png'
      };
    },
    onLoad(options) {
      this.queryobj.query = options.query || ''
      this.queryobj.cid = options.cid || ''

      this.getGoodsList()
    },
    methods: {
      async getGoodsList(cb) {
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/goods/search', this.queryobj)
        //  cb不赋值默认会返回undefined，undefined为假，不执行&&右边的语句；当cb是个函数时，cb打印出来是个函数，有值，为真，执行&&右边语句cb(),cb()意思是执行cb函数
        cb && cb()

        if (res.meta.status !== 200) return uni.$showMsg()
        this.goodsList = [...this.goodsList, ...res.message.goods]
        this.total = res.message.total
      },
      gotoDetail(item) {
        uni.navigateTo({
          url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
        })
      }
    },
    //uni提供的触底事件函数
    onReachBottom() {
      //判断数据是否加载完毕
      if (this.queryobj.pagenum * this.queryobj.pagesize >= this.total) return uni.$showMsg('数据加载完毕！')

      //页码自增+1
      this.queryobj.pagenum++
      //触底后再次获取
      this.getGoodsList()
    },
    onPullDownRefresh() {
      //重置关键数据
      this.queryobj.pagenum = 1
      this.total = 0
      this.goodsList = []
      //重新发起数据请求，并且关闭下拉刷新效果
      this.getGoodsList(() => uni.stopPullDownRefresh())
    }
  }
</script>

<style lang="scss">

</style>