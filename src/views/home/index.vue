<template>
  <div>
    <!-- 导航头 -->
    <van-nav-bar
      fixed
      title="黑马头条"
    />
    <!-- 频道列表 -->
    <van-tabs animated v-model="activeIndex">
      <!-- 遍历标签页，显示频道列表 -->
      <van-tab
        v-for="channel in channels"
        :title="channel.name"
        :key="channel.id">

        <!-- 下拉加载更多组件 -->
        <van-pull-refresh
          :success-text="successText"
          v-model="currentChannel.pullLoading"
          @refresh="onRefresh">
          <!-- 文章列表,不同的标签页下有不同的列表 -->
          <van-list
            v-model="currentChannel.loading"
            :finished="currentChannel.finished"
            finished-text="没有更多了"
            @load="onLoad"
          >
            <van-cell
              v-for="article in currentChannel.articles"
              :key="article.art_id.toString()"
              :title="article.title"
            >
              <div slot="label">
                <!-- grid 显示封面
                  article.cover.type   0 没有图片   1 1个图片 3 3个图片
                 -->
                <van-grid v-if="article.cover.type" :border="false" :column-num="3">
                  <van-grid-item
                    v-for="(img, index) in article.cover.images"
                    :key="img + index"
                  >
                    <van-image height="80" :src="img" />
                  </van-grid-item>
                </van-grid>
                <p>
                  <span>{{ article.aut_name }}</span>&nbsp;
                  <span>{{ article.comm_count }}评论</span>&nbsp;
                  <span>{{ article.pubdate }}</span>&nbsp;

                  <van-icon name="cross" class="close" />
                </p>
              </div>
            </van-cell>
          </van-list>
        </van-pull-refresh>
      </van-tab>
    </van-tabs>
    
  </div>
</template>

<script>
import { getDefaultOrUserChannels } from '@/api/channel'
import { getArticles } from '@/api/article'

export default {
  name: 'Home',
  data() {
    return {
      // // 列表用的数据
      // list: [],
      // loading: false,
      // finished: false,
      // 频道列表
      channels: [],
      // tab是组件中默认显示的tab项的索引
      // 通过该index，可以找到当前的频道对象
      activeIndex: 0,
      // 下拉更新完毕之后显示，成功的提示
      successText: ''
    };
  },
  created () {
    // 加载频道列表
    this.loadChannels()
  },
  computed: {
    // 返回当前的频道对象
    currentChannel () {
      return this.channels[this.activeIndex]
    }
  },
  methods: {
    // 加载频道列表
    async loadChannels () {
      try {
        const data = await getDefaultOrUserChannels()
        // console.log(data)
        // 给所有的频道设置，时间戳和文章数组
        data.channels.forEach((channel) => {
          channel.timestamp = null
          channel.articles = []
          // 上拉加载
          channel.loading = false
          channel.finished = false
          // 下拉加载
          channel.pullLoading = false
        })
        this.channels = data.channels
      } catch (err) {
        console.log(err)
      }
    },
    // list组件的load
    async onLoad() {
      // 发送请求
      // 获取当前频道对象  --- 下面不需要写了，因为设置了一个当前频道的计算属性
      // const currentChannel = this.channels[this.activeIndex]
      //  当前频道对象 时间戳
      //  当前频道对象 文章数组
      const data = await getArticles({
        // 频道的id
        channel_id: this.currentChannel.id,
        // 时间戳
        timestamp: this.currentChannel.timestamp || Date.now(),
        // 是否包含置顶1，0不包含
        with_top: 1
      })

      // 记录文章列表，记录最后一条数据的时间戳
      this.currentChannel.timestamp = data.pre_timestamp
      // [[], []]
      this.currentChannel.articles.push(...data.results)
      // this.loading = false
      this.currentChannel.loading = false
      // 文章加载完毕
      // 如果某一个频道加载完毕，其它频道中的finished 也是加载完毕
      if (data.results.length === 0) {
        // this.finished = true
        this.currentChannel.finished = true
      }
    },
    // 下拉加载更多
    async onRefresh() {
      try {
        const data = await getArticles({
          // 频道的id
          channel_id: this.currentChannel.id,
          // 时间戳
          timestamp: Date.now(),
          // 是否包含置顶1，0不包含
          with_top: 1
        })

        // 设置加载完毕
        this.currentChannel.pullLoading = false
        // 把数据放到数组的最前面（最新数据）
        this.currentChannel.articles.unshift(...data.results)
        this.successText = `加载了${data.results.length}条数据`
      } catch (err) {
        console.log(err)
      }
    }
  }
}
</script>

<style lang="less" scoped>
// 在scoped中书写的样式，动态生成的标签或者子组件中不可用
// 深度作用选择器   /deep/
// .van-tabs /deep/ .van-tabs__content {
//   margin-top: 46px;
//   margin-bottom: 50px;
// }
// .van-tabs {
//   margin-top: 46px;
//   margin-bottom: 50px;
// }

.van-tabs {
  /deep/ .van-tabs__wrap {
    position: fixed;
    top: 46px;
    left: 0;
    z-index: 100;
  }
  /deep/ .van-tabs__content {
    margin-top: 90px;
    margin-bottom: 50px;
  }
}
.close {
  float: right;
}
</style>