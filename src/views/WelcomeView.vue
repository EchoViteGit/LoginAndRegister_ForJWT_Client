<script setup>

</script>

<template>
  <div style="width: 100vw;height: 100vh;overflow: hidden;display: flex">
    <div style="flex: 1;background-color: black">
      <el-image style="width: 100%;height: 100%;" fit="cover"
          src="src/background.png"/>
    </div>
    <div class="welcome-title">
      <div style="font-size: 40px;font-weight: bold">欢迎来到vue学习平台</div>
      <div style="margin-top: 10px">此项目为基于JWT校验方案的前后端分离平台登录项目</div>
      <div style="margin-top: 5px">使用技术栈：Vue3+SpringBoot3+MySQL8.0.33+Redis7.0.12+RabbitMQ</div>
    </div>
    <div class="right-card">
      <router-view v-slot="{Component}">
        <transition name="el-fade-in-linear" mode="out-in">
          <component :is="Component"/>
        </transition>
      </router-view>
    </div>
  </div>
</template>

<style scoped>
  .right-card{
    width: 400px;
    z-index: 1;
    background-color: var(--el-bg-color);
  }
  .welcome-title{
    position: absolute;
    bottom: 30px;
    left: 30px;
    color: white;
    text-shadow: 0 0 10px black;
    font-size: 20px;
  }
</style>
