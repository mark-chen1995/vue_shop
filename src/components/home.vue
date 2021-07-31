<template>
  <div>
    <el-button type="info" @click="logout">退出</el-button>
  </div>
</template>

<script>
export default {
  data () {
    return {}
  },
  components: {},
  methods: {
    logout: function () {
      // 清空sessionStorage中保存的token
      sessionStorage.clear()
      // 重定向到登录页
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped></style>
