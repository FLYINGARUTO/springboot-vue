<script setup>

import {reactive,ref} from "vue";
import router from "../../router/router.js";
import {User,Message,Lock,Check} from "@element-plus/icons-vue"
import {get,post} from "../../net/index.js"
import {ElMessage} from "element-plus";

const form=reactive({
  username:"",
  password:"",
  password_repeat:"",
  email:"",
  code:""
})

const formRef=ref()


//输入校验validator
const validateUsername=(rule,value,callback)=>{
  if(value==='')
    callback(new Error("用户名不能为空!"))
  else if(!/^[a-z0-9A-Z\u4e00-\u9fa5]+$/.test(value))
    callback(new Error("用户名不能包含特殊字符！"))
  else
    callback()

}
const confirmPassword=(rule,value,callback)=>{

  if(value==='')
    callback(new Error("请再次输入密码"))
  else if(value!==form.password)
    callback(new Error("两次输入不一致"))
  else
    callback()
}
const rule={
  username:[
    {validator:validateUsername,trigger:["blur"]},
    {min:1,max:10,message:"用户名为1～10位"}
  ],password:[
    {required:true,trigger: ["blur"], message: "密码不能为空！"},
    {min:6,max:20,message: "密码为6～20位"}
  ],password_repeat:[
    {validator:confirmPassword,trigger: ["blur"]},
    {min:6,max:20,message: "密码为6～20位"}
  ],email:[
    {required:true,trigger: ["blur"], message: "电子邮箱不能为空！"}
  ],code:[
    {required:true,trigger: ["blur"], message: "验证码不能为空！"},
    {min:6,max:6,message: "验证码为6位"}
  ]
}

const getCode=()=>{
  if(/^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/.test(form.email)){
    get(`/api/auth/getVerifyCode?type=register&email=${form.email}`, ()=>{
      ElMessage.success(`验证码已发至${form.email}，请注意查收`)
    },(message)=>{
      ElMessage.error(message)
    })
  }else {
    ElMessage.warning("电子邮箱格式有误，请重新输入")
  }
}

const submitRegisterForm=()=>{
  post('/api/auth/register',{...form},()=>{
    ElMessage.success("注册成功，即将跳转")
    router.push('/')
  })
}


</script>

<template>
<div style="margin: 0 20px;text-align: center">
  <div style="margin-top: 140px;font-size: 30px;font-weight: bold">注册</div>
  <div style="font-size: 13px;color: grey">完善信息后，请求验证码以完成注册</div>
  <div style="margin-top: 50px">
    <el-form :model="form" ref="formRef" :rules="rule">
      <el-form-item prop="username">
        <el-input v-model="form.username" maxlength="10" placeholder="用户名" type="text">
          <template #prefix>
            <el-icon><User/></el-icon>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input v-model="form.password" maxlength="20" placeholder="密码" type="password">
          <template #prefix>
            <el-icon><Lock/></el-icon>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="password_repeat">
        <el-input v-model="form.password_repeat" maxlength="20" placeholder="确认密码" type="password">
          <template #prefix>
            <el-icon><Check/></el-icon>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="email">
        <el-input v-model="form.email"  placeholder="电子邮箱" type="text">
          <template #prefix>
            <el-icon><Message/></el-icon>
          </template>
        </el-input>
      </el-form-item>
      <el-form-item prop="code">
        <el-row :gutter="20" style="width: 100%">
          <el-col :span="17" style="left: 0">
            <el-input v-model="form.code" placeholder="验证码" type="text"></el-input>
          </el-col>
          <el-col :span="7" style="right: 0">
            <el-button @click="getCode" type="primary" plain>获取验证码</el-button>
          </el-col>
        </el-row>
      </el-form-item>
      <div style="margin-top: 30px;">
        <el-button @click="submitRegisterForm" type="success" plain style="width: 150px;">立即注册</el-button>
      </div>
      <el-divider style="font-size: 13px;color:grey">已经有账号？⬇️</el-divider>
      <div style="margin-top: 30px;">
        <el-button @click="router.push('/')" type="warning" plain style="width: 150px;">立即登录</el-button>
      </div>
    </el-form>
  </div>
</div>
</template>

<style scoped>

</style>