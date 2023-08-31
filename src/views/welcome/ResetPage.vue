<script setup>
import {computed, reactive, ref} from "vue";
import {EditPen, Lock, Message} from "@element-plus/icons-vue";
import {get,post} from "@/net";
import {ElMessage} from "element-plus";
import router from "@/router";

const active = ref(0)
const coldTime = ref(0)
const formRef = ref()

const  form = reactive({
  email:'',
  code:'',
  password:'',
  password_repeat:''
})

function askCode(){
  coldTime.value = 60
  if(isEmailValid){
    get(`/api/auth/ask-code?email=${form.email}&type=reset`,()=>{
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

const isEmailValid = computed(()=>/^[\w.-]+@[\w.-]+\.\w+$/.test(form.email))

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

function confirmReset(){
  formRef.value.validate((valid)=>{
    if(valid){
      post("/api/auth/reset-confirm",{
        email:form.email,
        code: form.code
      },()=>active.value++)
    }
  })
}

function deRest(){
  formRef.value.validate((valid)=>{
    if(valid){
      post("/api/auth/reset-password",{...form},()=>{
        ElMessage.success('密码重置成功，请重新登录')
        router.push('/')
      })
    }
  })
}
</script>

<template>
  <div style="text-align: center">
    <div style="margin-top: 80px">
      <el-steps align-center finish-status="success" :active="active">
        <el-step title="验证电子邮件"/>
        <el-step title="重新设置密码"/>
      </el-steps>
    </div>
    <div v-if="active === 0" style="margin: 0 30px">
      <div style="margin-top: 80px">
        <div style="font-size: 25px;font-weight: bold">重置密码</div>
        <div style="font-size: 14px;color: grey">请输入需要充值密码的电子邮件地址</div>
      </div>
      <div style="margin-top: 50px;">
        <el-form :model="form" :rules="rule" ref="formRef">
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
        <el-button style="width: 270px" type="warning" @click="confirmReset" plain>开始重置密码</el-button>
      </div>
    </div>
    <div v-if="active === 1" style="margin: 0 30px">
      <div style="margin-top: 80px">
        <div style="font-size: 25px;font-weight: bold">重置密码</div>
        <div style="font-size: 14px;color: grey">请填写你的新密码</div>
      </div>
      <div style="margin-top: 50px">
        <el-form :model="form" :rules="rule" ref="formRef">
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
        </el-form>
      </div>
      <div style="margin-top: 80px">
        <el-button style="width: 270px" type="danger" @click="deRest" plain>立即重置密码</el-button>
      </div>
    </div>
    <div style="margin-top: 100px">
      <el-button style="width: 270px" type="warning" @click="router.push('/')" plain>返回登录页</el-button>
    </div>
  </div>
</template>

<style scoped>

</style>
