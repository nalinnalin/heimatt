<template>
  <div class="home-container">
    <van-nav-bar title="主页" fixed />
    <van-tabs sticky :offset-top="46" v-model="active">
      <van-tab v-for="item in channels" :title="item.name" :key="item.id">
        <van-pull-refresh v-model="item.isLoading" @refresh="onRefresh">
          <van-list
            v-model="item.loading"
            :finished="item.finished"
            finished-text="没有更多了"
            @load="onLoad"
          >
            <van-cell v-for="subitem in item.articleList" :key="subitem.art_id">
              <template #title>
                <!-- 标题 -->
                <h4>{{ subitem.title }}</h4>
                <!-- 图片 -->
                <van-grid v-if="subitem.cover.type !== 0" :column-num="3">
                  <van-grid-item
                    v-for="(imgItem, index) in subitem.cover.images"
                    :key="index"
                  >
                    <van-image lazy-load :src="imgItem" />
                  </van-grid-item>
                </van-grid>
                <!-- 内容 -->
                <div class="content">
                  <div class="left">
                    <span>{{ subitem.aut_name }}</span>
                    <span>{{ subitem.comm_count }}评论</span>
                    <span>{{ subitem.pubdate | fromTime }}</span>
                  </div>
                  <div class="right">
                    <van-icon name="cross" @click="showMore(subitem.art_id)" />
                  </div>
                </div>
              </template>
            </van-cell>
          </van-list>
        </van-pull-refresh>
      </van-tab>
    </van-tabs>
    <div class="more" @click="showChannel">
      <van-icon name="bars" />
    </div>
    <!-- 频道弹出组件 -->
    <channel ref="channelRef" :channels="channels" />
    <!-- 更多子组件 -->
    <more ref="moreRef" @dislike="dislike" />
  </div>
</template>

<script>
import { getChannels } from '@/api/channel'
import { getArticleListById } from '@/api/article'
import channel from './components/channel'
import more from './components/more'
export default {
  name: 'Home',
  components: {
    channel,
    more
  },
  data () {
    return {
      active: 0, // 第几个选中了
      channels: [] // 频道数组
    }
  },
  mounted () {
    this.getChannelsData()
  },
  methods: {
    /**
     * 获取频道列表
     */
    async getChannelsData () {
      const res = await getChannels()

      // 在channels中设置文章列表
      res.data.channels.forEach(item => {
        // item.articleList = [] // 文章列表
        // item.loading = false // 正在加载中
        // item.isLoading = false // 下拉刷新
        // item.finished = false // 加载完毕了
        // 变成响应式
        this.$set(item, 'articleList', [])
        this.$set(item, 'loading', false)
        this.$set(item, 'isLoading', false)
        this.$set(item, 'finished', false)
      })

      this.channels = res.data.channels
    },
    // 上拉加载触发
    async onLoad () {
      const channel = this.channels[this.active]

      const res = await getArticleListById(channel.id)

      if (res.data.results.length === 0) {
        channel.articleList = []
        // 没有更多数据啦...
        channel.finished = true
      } else {
        channel.articleList = [...channel.articleList, ...res.data.results]
        channel.loading = false
        channel.isLoading = false
        channel.finished = false
      }
    },
    // 下拉刷新
    onRefresh () {},
    showChannel () {
      this.$refs.channelRef.show = true
    },
    // 显示更多子组件
    showMore (artId) {
      this.$refs.moreRef.artId = artId
      this.$refs.moreRef.show = true
    },
    // 对文章不喜欢
    dislike (artId) {
      // 获取当前频道的文章列表
      const currentAticleList = this.channels[this.active].articleList

      // 找到不喜欢文章在当前文章列表中的索引
      const index = currentAticleList.findIndex(item => item.art_id === artId)

      // 删除
      currentAticleList.splice(index, 1)
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  position: relative;
}
.van-nav-bar {
  background-color: #3e9df8;
  ::v-deep .van-nav-bar__title {
    color: #fff;
  }
}
.van-tabs__content {
  padding-top: 46px;
}
.more {
  position: absolute;
  right: 0;
  top: 46px;
  height: 44px;
  width: 10%;
  z-index: 100;
  background-color: pink;
  text-align: center;
}
.van-icon-bars {
  font-size: 20px;
  width: 100%;
  height: 44px;
  line-height: 44px;
}
/deep/ .van-tabs__wrap {
  width: 90%;
}
.van-cell {
  height: 200px;
}
.content {
  display: flex;
  justify-content: space-between;
  .left {
    span {
      margin-right: 10px;
    }
  }
  .right {
  }
}
</style>
