<template>
  <van-list
    v-model="loading"
    :finished="finished"
    finished-text="没有更多了"
    @load="onLoad"
  >
    <van-cell
      v-for="item in list"
      :key="item.id.toString()"
      :title="item.name"
    />
  </van-list>
</template>

<script>
import { getFollowings, getFollowers } from '@/api/user'
export default {
  name: 'UserList',
  props: ['type'],
  data () {
    return {
      list: [],
      loading: false,
      finished: false,
      page: 1,
      per_page: 10
    }
  },
  methods: {
    async onLoad () {
      try {
        let data = null
        if (this.type === 0) {
          // 获取关注用户
          data = await getFollowings({
            page: this.page,
            per_page: this.per_page
          })
        } else {
          // 获取粉丝
          data = await getFollowers({
            page: this.page,
            per_page: this.per_page
          })
        }
        this.page++
        this.list.push(...data.results)
        this.loading = false
        if (data.results.length === 0) {
          this.finished = true
        }
      } catch (err) {
        this.$toast.fail('加载数据失败')
      }
    }
  }
}
</script>

<style>
</style>
