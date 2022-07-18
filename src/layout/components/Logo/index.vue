<template>
  <div class="logo">
    <!-- <img src="~@/assets/images/logo.png" alt="" :class="{ 'mr-2': !collapsed }" /> -->
    <h2 v-show="!collapsed" class="title">4K路内智能相机</h2>
  </div>
</template>

<script>
  export default {
    name: 'Index',
    props: {
      collapsed: {
        type: Boolean,
      },
    },
  };
</script>

<style lang="less" scoped>
  .logo {
    display: flex;
    align-items: center;
    justify-content: left;
    height: 64px;
    line-height: 64px;
    overflow: hidden;
    white-space: nowrap;
    padding-left: 20px;
    border-right: 1px solid #28364a;
    border-bottom: 1px solid #43546d;
    background: #364760;
    font-size: 16px;

    img {
      width: auto;
      height: 32px;
    }

    .title {
      font-size: 16px;
      color: #fff;
      margin-top: -4px;
      font-weight: bold;
      margin-bottom: 0;
    }
  }
</style>
