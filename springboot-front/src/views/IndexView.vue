<script setup>
  import {logout,getAccessToken} from "../net/index.js";
  import router from "../router/router.js";

  function userLogout(){
    //如果token为null，直接返回登录页
    if(getAccessToken()===null)   router.push('/')
    //否则调用logout业务代码
    else{
      logout(()=>{
        router.push('/')
      })
    }
  }
</script>

<template>
  <el-button @click="userLogout">退出登录</el-button>
</template>

<style scoped>

</style>