<template>
  <view>
    <view class="search-box">
      <uni-search-bar @confirm="search" @input="input" :radius="100" :cancel-button="auto"
        :focus="true"></uni-search-bar>
    </view>

    <!-- 搜索建议列表 -->
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>

    <!-- 搜索历史 -->
    <view class="history-box" v-else>
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click="clean"></uni-icons>
      </view>
      <!-- 列表区域 -->
      <view class="history-list">
        <uni-tag :text="item" v-for="(item, i) in histories" :key="i"
          :customStyle="'margin-top: 5px;margin-right: 5px;'" @click="gotoGoodsList(item)"></uni-tag>

      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        //定义一个空的计时器
        timer: null,
        //输入的内容
        kw: '',
        //搜索结果列表
        searchResults: [],
        //搜索历史组件
        historyList: []
      };
    },
    onLoad() {
      this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
    },
    methods: {
      input(e) {
        clearTimeout(this.timer)
        this.timer = setTimeout(() => {
          this.kw = e
          this.getSearchList()
        }, 500)
      },

      async getSearchList() {
        //判断搜索关键词是否为空
        if (this.kw.length === 0) {
          this.searchResults = []
          return
        }

        //解构出data赋值给res
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/goods/qsearch', {
          query: this.kw
        })

        if (res.meta.status !== 200) return uni.$showMsg()
        this.searchResults = res.message

        this.saveSearchHistory()
      },

      gotoDetail(item) {
        uni.navigateTo({
          url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
        })
      },
      saveSearchHistory() {

        const set = new Set(this.historyList)
        set.delete(this.kw)
        set.add(this.kw)

        this.historyList = Array.from(set)

        //数据持久化
        uni.setStorageSync('kw', JSON.stringify(this.historyList))
      },
      clean() {
        this.historyList = []
        uni.setStorageSync('kw', '[]')
      },
      gotoGoodsList(kw) {
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?query=' + kw
        })
      }
    },
    computed: {
      histories() {
        return [...this.historyList].reverse()
      }
    }
  }
</script>

<style lang="scss">
  .search-box {
    position: sticky;
    top: 0;
    z-index: 999;
  }

  /deep/ .uni-searchbar {
    /* #ifndef APP-NVUE */
    display: flex;
    /* #endif */
    flex-direction: row;
    position: relative;
    padding: 10px;
    background-color: #C00000;
  }

  .sugg-list {
    padding: 0 10px;

    .sugg-item {
      padding: 13px 0;
      font-size: 12px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      border: 1px solid #efefef;

      .goods-name {
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        margin-right: 3px;
      }
    }
  }

  .history-box {
    padding: 0 5px;

    .history-title {
      display: flex;
      // 两端分别贴边
      justify-content: space-between;
      height: 40px;
      align-items: center;
      font-size: 13px;
      border-bottom: 1px solid #efefef;

      .history-list {
        display: flex;
        // 允许换行
        flex-wrap: wrap;

      }
    }
  }
</style>