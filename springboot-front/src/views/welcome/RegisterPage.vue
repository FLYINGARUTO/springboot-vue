<script setup>
  import {reactive,ref} from "vue";
  import {Check, Lock, Message, User} from "@element-plus/icons-vue";
  import router from "../../router/router.js";
  import {get,post} from "../../net/index.js"
  import {ElMessage} from "element-plus";

  const formRef=ref()
  let cooldown=ref(0)
  const form=reactive({
    username:"",
    password:"",
    password_repeat:"",
    code:"",
    email:""
  })
  //提交注册
  const register=()=>{
    formRef.value.validate((valid)=>{
      if(valid){
          post("/api/auth/register",{...form},()=>{
            ElMessage.success("注册成功，跳转至登录页～")
            router.push('/')
          })
      }else{
        ElMessage.warning("请完善表单内容")
      }
    })
  }
  //验证username框输入
  const validateUsername=(rule,value,callback)=>{
    if(value==='')
      callback(new Error("请输入用户名"))
    else if(!/^[a-zA-Z0-9\u4e00-\u9fa5]+$/.test(value))
      callback(new Error("用户名不能包含特殊字符"))
    else
      callback()
  }
  //判断第二次输入的密码
  const confirmPassword=(rule,value,callback)=>{
    if(value==='')
      callback(new Error("请再次输入密码"))
    else if(value!==form.password)
      callback(new Error("两次输入不一致，请重新检查"))
    else
      callback()
  }


  //获取验证码
  const getCode=()=>{
    if(/^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/.test(form.email)){
      get(`/api/auth/getVerifyCode?type=register&email=${form.email}`,
          ()=>{
                  cooldown.value=60
                  ElMessage.success(`验证码已发送至${form.email}`)
                  //setInterval控制每1秒冷却时间减一
                  //当冷却结束，即cooldown===1时，用clearInterval结束Interval的使命
                  let intervalId=setInterval(()=>{
                    cooldown.value--
                    if(cooldown.value===0){
                      clearInterval(intervalId)
                      ElMessage.success("冷却结束，可重新获取验证码")
                    }

                  },1000)

              },
          (message)=>{
              ElMessage.warning(message)
              cooldown.value=0
          }
          )
    }else{
      ElMessage.warning("邮箱地址格式不符合规范，请重新输入")
    }

  }

  const rule={
    username:[
      {validator:validateUsername,trigger:['blur']
      },{ min: 1,max:10,message:"字符个数要求：1～10"}
    ],
    password:[
      {required: true,message:"请输入密码",trigger:['blur']},
      {min:6,max:20,message:"字符个数要求：6~20"}
    ],
    password_repeat:[
      {validator:confirmPassword,trigger:['blur']},
      {min:6,max:20,message:"字符个数要求：6~20"}
    ],
    email:[
      {required: true,message:"请输入电子邮箱",trigger:['blur']},
      {min:5,message:"字符个数要求：>=5"}
    ],
    code:[
      {required: true,message:"请输入验证码",trigger:['blur','change']},{
      min:6,max:6,message:"验证码为6位"
      }
    ]
  }
</script>

<template>
  <div style="text-align:center;margin: 0 20px">
    <div style="margin-top: 140px">
      <div style="font-size: 30px;font-weight: bold">注册</div>
      <div style="font-size: 13px;color: grey">完善信息以完成注册</div>
    </div>
    <div style="margin-top: 50px">
      <el-form :model="form" :rules="rule" ref="formRef">
        <el-form-item prop="username">
          <el-input v-model="form.username" maxlength="10" placeholder="用户名" type="text">
            <template #prefix>
              <el-icon><User/></el-icon>
            </template>
          </el-input>
        </el-form-item>

        <el-form-item prop="password">
          <el-input v-model="form.password" placeholder="密码" maxlength="20" type="password">
            <template #prefix>
              <el-icon><Lock/></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="password_repeat">
          <el-input v-model="form.password_repeat" placeholder="确认密码"  maxlength="20" type="password">
            <template #prefix>
              <el-icon><Check/></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="email">
          <el-input v-model="form.email" placeholder="邮箱" type="text">
            <template #prefix>
              <el-icon><Message /></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="code">
          <el-row :gutter="20" style="width: 100%;">
            <el-col :span="17">
              <el-input v-model="form.code" placeholder="验证码" type="text" maxlength="6"/>
            </el-col>
            <el-col :span="7">
              <el-button @click="getCode" type="primary" :disabled="cooldown>0" plain>
                {{   cooldown>0   ?    `${cooldown}s`  :  "获取验证码" }}
              </el-button>
            </el-col>
          </el-row>
        </el-form-item>

        <div style="margin-top: 40px">
          <el-button @click="register" style="width: 270px;" type="success" plain>注册</el-button>
        </div>
        <el-divider>已有密码？⬇️</el-divider>
        <el-button @click="router.push('/')" style="width: 270px;" type="warning" plain>登录页</el-button>

      </el-form>
    </div>

  </div>
</template>

<style scoped>

</style>