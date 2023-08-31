<script setup>

import {EditPen, Lock, Message, User} from "@element-plus/icons-vue";
import router from "@/router";
import {computed, reactive, ref} from "vue";
import {get,post} from "@/net";
import {ElMessage} from "element-plus";

const coldTime = ref(0)
const formRef = ref()

const form = reactive({
  username:'',
  password:'',
  password_repeat:'',
  email:'',
  code:''
})

const validateUsername = (rule,value,callback) =>{
  if(value === ''){
    callback(new Error('请输入用户名'))
  }else if(!/^[a-zA-Z0-9\u4e00-\u9fa5]+$/.test(value)){
    callback(new Error('用户名不能包含特殊字符,仅可使用中/英文'))
  }else {
    callback()
  }
}

const validatePasswordRepeat = (ule,value,callback)=>{
  if(value === '')
    callback(new Error('请再次输入密码'))
  else if(value!==form.password)
    callback(new Error('两次输入密码不一致'))
  else
    callback()
}

const validatePassword = (ule,value,callback)=>{
  if(value === '')
    callback(new Error('请输入密码'))
  else if(!/^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z\\W]{6,20}$/.test(form.password)){
    callback(new Error('密码必须包含字母和数字'))
  }else {
    callback()
  }
}

const rule = {
  username:[
    {validator:validateUsername,trigger:['blur','change']},
    {min: 4,max: 10,message: '用户名的长度再4-10个字符'}
  ],
  password:[
    {validator:validatePassword,trigger:['blur','change']},
    {min:6,max:20,message: '密码长度在6-20字符之间',trigger:['blur','change']}
  ],
  password_repeat:[
    {validator:validatePasswordRepeat,trigger:['blur','change']},
  ],
  email:[
    {required: true,message: '请输入邮件地址',trigger:['blur']},
    {type: 'email',message: '请输入合法邮箱地址',trigger:['blur','change']}
  ],
  code:[
    {required: true,message: '请输入邮件验证码',trigger:['blur']},
    {min:6,max:6,message: '验证码长度6位',trigger:['blur']}
  ]
}

function askCode(){
  coldTime.value = 60
  if(isEmailValid){
    get(`/api/auth/ask-code?email=${form.email}&type=register`,()=>{
      ElMessage.success(`验证码已发送至:${form.email},请注意查收！`)
      const interval = setInterval(() => {
            if (coldTime.value-- === 1) {
              clearInterval(interval);
            }
          }, 1000
      );
    },(message)=>{
      ElMessage.warning(message)
      coldTime.value = 0
    })
  }else {
    ElMessage.warning('请输入正确的电子邮件地址')
  }
}

const isEmailValid = computed(()=>{
  return /^[\w.-]+@[\w.-]+\.\w+$/.test(form.email)
})

function register(){
  formRef.value.validate((valid)=>{
    if(valid){
      post('/api/auth/register',{
        ...form
      },()=>{
        ElMessage.success('注册成功')
        router.push('/')
      })
    }else {
      ElMessage.warning('请完成注册表单')
    }
  })
}

</script>

<template>
  <div style="text-align:center;margin: 0 20px">
    <div style="margin-top: 100px">
      <div style="font-size:25px;font-weight: bold">注册新用户</div>
      <div style="font-size:14px;color: grey;margin-top: 20px">欢迎注册此平台，请填写信息完成注册</div>
    </div>
    <div style="margin-top: 50px">
      <el-form :model="form" :rules="rule" ref="formRef">
        <el-form-item prop="username">
          <el-input v-model="form.username" maxlength="10" type="text" placeholder="用户名">
            <template #prefix>
              <el-icon><User/></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="form.password" maxlength="20" type="password" placeholder="密码">
            <template #prefix>
              <el-icon><Lock/></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="password_repeat">
          <el-input v-model="form.password_repeat" maxlength="20" type="password" placeholder="重复密码">
            <template #prefix>
              <el-icon><Lock/></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="email">
          <el-input v-model="form.email" type="email" placeholder="电子邮件地址">
            <template #prefix>
              <el-icon><Message/></el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="code">
          <el-row :gutter="10" style="width: 100%">
            <el-col :span="17">
              <el-input v-model="form.code" maxlength="6" type="text" placeholder="请输入验证码">
                <template #prefix>
                  <el-icon><EditPen/></el-icon>
                </template>
              </el-input>
            </el-col>
            <el-col :span="5">
              <el-button type="success" :disabled="coldTime||!isEmailValid" @click="askCode">
                {{ coldTime > 0 ? `请稍后 ${coldTime} 秒` : '获取验证码' }}
              </el-button>
            </el-col>
          </el-row>
        </el-form-item>
      </el-form>
    </div>
    <div style="margin-top: 80px">
      <el-button style="width: 270px" type="warning" @click="register" plain>立即注册</el-button>
    </div>
    <div style="margin-top: 20px">
      <span style="font-size: 14px;line-height: 15px;color: grey">已有帐号？</span>
      <el-link style="translate: 0 -2px" @click="router.push('/')">立即登录</el-link>
    </div>
  </div>
</template>

<style scoped>

</style>
