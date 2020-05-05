<template>
  <div class="login-container">
       <!--
      el-form 表单组件
      每个表单项都必须使用 el-form-item 组件包裹
     -->
    <div class="login-form-wrap">
      <div class="login-head">
        <div class="logo"></div>
    </div>
     <!--
        配置 Form 表单验证：
        1、必须给 el-from 组件绑定 model 为表单数据对象
        2、给需要验证的表单项 el-form-item 绑定 prop 属性
           注意：prop 属性需要指定表单对象中的数据名称
        3、通过 el-from 组件的 rules 属性配置验证规则
       -->

      <!--   手动触发表单验证：
        1、给 el-form 设置 ref 起个名字（随便起名，不要重复即可）
        2、通过 ref 获取 el-form 组件，调用组件的 validate 进行验证
       -->
    <el-form class="login-form" ref="login-form" :model="user" :rules="formRules">
      <el-form-item prop="mobile">
        <el-input placeholder="请输入手机号111" v-model="user.mobile"></el-input>
      </el-form-item>
      <el-form-item prop="code">
        <el-input placeholder="请输入验证码111" v-model="user.code"></el-input>
      </el-form-item>
       <el-form-item prop="agree">
        <el-checkbox v-model="user.agree">我已阅读并同意用户协议和隐私条款</el-checkbox>
      </el-form-item>
      <el-form-item>
        <el-button class="login-btn" type="primary"  :loading="loginLoding" @click="onLogin">登录</el-button>
      </el-form-item>
    </el-form>
  </div>
</div>
</template>

<script>
// 加载user模块中的方法login方法
import { login } from '@/api/user'
export default {
  name: 'LoginIndex',
  data () {
    return {
      user: {
        mobile: '13911111111', // 手机号
        code: '246810', // 验证码
        agree: false // 是否同意协议
      },
      // checked: true, // 是否同意协议的选中状态
      loginLoding: false, // 登录的 loading 状态
      formRules: { // 表单验证规则配置
      // 要验证的数据名称：规则列表[]
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'change' },
          { pattern: /^1[3|5|7|8|9]\d{9}$/, message: '请输入正确的号码格式', trigger: 'change' }
        ],
        code: [
          { required: true, message: '验证码不能为空', trigger: 'change' },
          { pattern: /^\d{6}$/, message: '请输入正确的验证码格式' }
        ],
        // 是否同意协议的校验规则
        agree: [
          {
            // 自定义校验规则：https://element.eleme.cn/#/zh-CN/component/form#zi-ding-yi-xiao-yan-gui-ze
            // 验证通过：callback()
            // 验证失败：callback(new Error('错误消息'))
            validator: (rule, value, callback) => {
              if (value) {
                callback()
              } else {
                callback(new Error('请同意用户协议'))
              }
            },
            trigger: 'change'
          }
        ]
      }
    }
  },
  methods: {
    onLogin () {
    // 获取表单数据（根据接口要求绑定数据）
      // const user = this.user
      // 表单验证
      // validate 方法是异步的
      this.$refs['login-form'].validate(valid => {
        // 如果表单验证失败,停止请求提交
        if (!valid) {
          return
        }
        // 验证通过，请求登录
        this.login()
      })
    },
    login () {
      // 表单验证通过提交登录
      // 开启登陆中 loading...
      this.loginLoding = true
      login(this.user).then(res => {
        console.log(res)
        // 登录成功
        this.$message({
          message: '恭喜你，这是一条成功消息',
          type: 'success'
        })
        // 关闭loading
        this.loginLoding = false
        // 本地存储token,本地存储是字符串格式,给要存储的数据起名为user
        window.localStorage.setItem('user', JSON.stringify(res.data.data))
        // 跳转到首页
        this.$router.push({
          name: 'home'
        })
      }).catch(err => {
        console.log('登录失败', err)
        // 登录失败
        this.$message.error('错了哦，这是一条错误消息')
        // 关闭loading
        this.loginLoding = false
      })
    }
  }
}
</script>

<style>
.login-container{
 position: fixed;
 left: 0;
 top: 0;
 right: 0;
 bottom: 0;
 /* 水平排列 */
 display: flex;
 /* 垂直排列 */
 flex-direction: column;
 /* 垂直居中 */
 justify-content: center;
 /* 水平居中 */
 align-items: center;
 background: url("./login_bg.jpg") no-repeat;
 background-size: cover;
}
.login-form-wrap{
    min-width: 300px;
    padding: 30px 50px 10px;
    background-color: #fff;
}
/* logo */
.login-head{
  display: flex;
  justify-content: center;
}
.logo{
    width: 200px;
    height: 57px;
    background: url("./logo_index.png") no-repeat;
    background-size: contain;
}
/* 按钮 */
.login-btn {
      width: 100%;
 }

</style>
