<script setup>
import {reactive,ref} from "vue";
import {Back, Check, Lock, Message} from "@element-plus/icons-vue";
import router from "../../router/router.js";
import {ElMessage} from "element-plus";
import {get, post} from "../../net/index.js";
const active=ref(0)
const cooldown=ref(0)
const form=reactive({
  email:"",
  code:"",
  password:"",
  password_repeat:""
})

const formRef=ref()

//判断第二次输入的密码
const confirmPassword=(rule,value,callback)=>{
  if(value==='')
    callback(new Error("请再次输入密码"))
  else if(value!==form.password)
    callback(new Error("两次输入不一致，请重新检查"))
  else
    callback()
}
const rule={

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

const getCode=()=>{
  if(/^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/.test(form.email)){
    get(`/api/auth/getVerifyCode?type=reset&email=${form.email}`,()=>{
      cooldown.value=60
      ElMessage.success("验证码已发送至邮箱，请注意查收")
      let intervalID=setInterval(()=>{
        cooldown.value--
      },1000)
      if(cooldown.value===0)
        clearInterval(intervalID)
    },()=>{
      cooldown.value=0
      ElMessage.warning("发生未知异常")
    })

  }else{
    ElMessage.warning("邮箱格式不正确，请重试")
  }
}
const verify=()=>{
  if(form.code===""||form.email==="")
    ElMessage.warning("输入不能为空")
    return
  if(/^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/.test(form.email)){
    post("api/auth/resetConfirm",{
      "email":form.email,
      "code":form.code
    },()=>{
      active.value===0?active.value++ : active.value
      ElMessage.success("身份验证成功")
    },()=>{
      ElMessage.warning("验证出现异常")
    })
  }else{
    ElMessage.warning("邮箱格式不正确，请重试")
  }

}

const reset=()=>{
  formRef.value.validate((valid)=>{
    if(valid){
      post("/api/auth/reset-password",{
        "email":form.email,
        "password":form.password,
        "code":form.code
      },()=>{
        ElMessage.success("密码重置成功，请牢记")
        router.push("/")
      })
    }
  })
}

</script>

<template>

  <div style="text-align: center;margin: 0 20px">
    <el-page-header @back="router.push('/')" >
      <template #content>
        <span style="font-size: 14px;color:grey;"> 重置密码 </span>
      </template>
    </el-page-header>

    <div style="margin-top: 200px">
      <el-steps :active="active" finish-status="success" align-center>
        <el-step @click="active===1?active--:active" title="用户验证"/>
        <el-step title="密码重置"/>
      </el-steps>
    </div>
    <div style="margin-top: 20px" v-if="active===0">

      <div style="margin-top: 60px;">
        <el-form :model="form" :rules="rule">
          <el-form-item prop="email">
            <el-input v-model="form.email" placeholder="input email..." type="email">
              <template #prefix>
                <el-icon><Message/></el-icon>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item prop="code">
            <el-row :gutter="20" style="width: 100%;">
              <el-col :span="17">
                <el-input v-model="form.code" placeholder="verify code">
                  <template #prefix>
                    <el-icon><Check/></el-icon>
                  </template>
                </el-input>
              </el-col>
              <el-col :span="7">
                <el-button type="primary" plain @click="getCode" :disabled="cooldown>0">
                  {{   cooldown>0   ?    `${cooldown}s`  :  "获取验证码" }}
                </el-button>
              </el-col>
            </el-row>

          </el-form-item>
        </el-form>
        <el-button @click="verify" style="width: 120px;margin-top: 60px " type="success" plain>下一步</el-button>
      </div>
    </div>
    <div style="margin-top: 20px" v-if="active===1">
      <div style="margin-top: 60px;">
        <el-form :model="form" :rules="rule" ref="formRef">
          <el-form-item prop="password">
            <el-input v-model="form.password" placeholder="input password..." type="password">
              <template #prefix>
                <el-icon><Lock/></el-icon>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item prop="password_repeat">
            <el-input v-model="form.password_repeat" placeholder="confirm password..." type="password">
              <template #prefix>
                <el-icon><Check/></el-icon>
              </template>
            </el-input>
          </el-form-item>
        </el-form>

      </div>

      <div style="margin-top: 80px">
            <el-button @click="reset" style="width: 120px" type="success" plain >重置</el-button>

      </div>


    </div>

  </div>

</template>

<style scoped>

</style>