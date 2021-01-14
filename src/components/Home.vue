<template>
  <div>
    <el-button type="info" @click="signOut">退出</el-button>
  </div>
</template>
<script>
export default {
  data() {
    return {}
  },
  methods: {
    signOut: function() {
      window.sessionStorage.clear()
      console.log(this.$router)
      this.$router.push('/login')
    }
  }
}
</script>
<style lang="less" scoped></style>
