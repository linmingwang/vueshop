<template>
  <el-container>
    <!-- 头部区域 -->
    <el-header><el-button type="info" @click="logout">退出</el-button></el-header>
    <!-- 页面主体 -->
    <el-container>
      <!-- 侧边栏区域 -->
      <el-aside width="200px">Aside</el-aside>
      <!-- 头部区域 -->
      <el-main>Main</el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: 'Home',
  methods: {
    // 登出
    logout() {
      window.sessionStorage.removeItem('token')
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped>

</style>
