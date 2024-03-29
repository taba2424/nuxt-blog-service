<template>
  <section class="container posts-page">
    <div style="flex: 1">
      <el-card v-if="post">
        <div
          slot="header"
          class="clearfix">
          <h2>{{ post.title }}</h2>
          <small>by {{ post.user.id }}</small>
        </div>
        <p>{{ post.body }}</p>
        <no-ssr>
          <p class="text-right">
            <el-button
              v-if="isLiked"
              :disabled="!isLoggedIn"
              type="warning"
              round
              @click="unlike">
              <span class="el-icon-star-on" />
              <span>{{ post.likes.length }}</span>
            </el-button>
            <el-button
              v-else
              :disabled="!isLoggedIn"
              type="warning"
              round
              @click="like">
              <span class="el-icon-star-off" />
              <span>{{ post.likes.length }}</span>
            </el-button>
          </p>
        </no-ssr>
        <p class="text-right">{{ post.created_at | time }}</p>
      </el-card>
      <p>
        <nuxt-link to="/posts">&lt; 投稿一覧へ戻る</nuxt-link>
      </p>
    </div>
  </section>
</template>

<script>
import moment from '~/plugins/moment'
import { mapGetters, mapActions } from 'vuex'
import cloneDeep from 'lodash.clonedeep'

export default {
  async asyncData({ store, route }) {
    if (store.getters['posts/posts'].find(p => p.id === route.params.id)) {
      return
    }
    await store.dispatch('posts/fetchPosts')
  },
  filters: {
    time(val) {
      return moment(val).format('YYYY/MM/DD HH:mm:ss')
    }
  },
  computed: {
    post() {
      return this.posts.find(p => p.id === this.$route.params.id)
    },
    isLiked() {
      if (!this.user) return false
      return this.post.likes.find(l => l.user_id === this.user.id)
    },
    ...mapGetters(['user', 'isLoggedIn']),
    ...mapGetters('posts', ['posts'])
  },
  methods: {
    like() {
      if (!this.isLoggedIn) {
        return
      }
      const likePayload = { user: this.user, post: this.post }
      this.addLikeToPost(cloneDeep(likePayload))
      this.addLikeLogToUser(cloneDeep(likePayload))
    },
    unlike() {
      if (!this.isLoggedIn) {
        return
      }
      const likePayload = { user: this.user, post: this.post }
      this.removeLikeToPost(cloneDeep(likePayload))
      this.removeLikeLogToUser(cloneDeep(likePayload))
    },
    ...mapActions(['addLikeLogToUser', 'removeLikeLogToUser']),
    ...mapActions('posts', ['addLikeToPost', 'removeLikeToPost'])
  }
}
</script>

<style scoped>
.post-page .el-table__row {
  cursor: pointer;
}
</style>
