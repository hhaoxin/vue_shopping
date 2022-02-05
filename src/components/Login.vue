<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt />
      </div>
      <!-- 表单 -->
      <el-form
        :rules="rules"
        ref="formRef"
        :model="loginForm"
        label-width="70px"
        class="login_form"
      >
        <el-form-item label="用户名:" prop="username">
          <el-input v-model="loginForm.username" placeholder="请输入用户名" prefix-icon="el-icon-user"></el-input>
        </el-form-item>
        <el-form-item label="密码:" prop="password">
          <el-input
            show-password
            placeholder="请输入密码"
            prefix-icon="iconfont icon-3702mima"
            v-model="loginForm.password"
          />
        </el-form-item>
        <el-button type="primary" @click="submitLogin">登录</el-button>
        <el-button type="info" @click="resetLoginForm">重置</el-button>
      </el-form>
    </div>
  </div>
</template>
<script>
export default {
  name: 'login',
  data() {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      // 验证规则对象
      rules: {
        username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { min: 6, max: 15, message: '密码长度在6到15个字符', trigger: 'blur' }]
      }
    }
  },
  methods: {
    resetLoginForm() {
      this.$refs.formRef.resetFields()
    },
    submitLogin() {
      this.$refs.formRef.validate(async valid => {
        if (!valid) return;
        // 登录验证API接口,已在main.js配置请求的根路径
        const { data: res } = await this.$http.post('login', this.loginForm)
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('登录失败，请检查重新登录!')
        }
        this.$message.success('登录成功')
        // 将登录成功之后的token保存到客户端的sessionStorage中
        // 其他API接口，必须登录之后才能访问
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.replace('./home')
      })
    }
  }
}
</script>
<style lang="less" scoped>
// 外部一级div
.login_container {
  background-color: darkgrey;
  height: 100%;
}
// 二级div
.login_box {
  height: 300px;
  width: 450px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute; // 绝对定位
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%); // 横轴-50%，竖轴-50%
  box-shadow: 0 0 10px;
}
.avatar_box {
  height: 130px;
  width: 130px;
  border-radius: 50%;
  border: 1px solid #eee;
  padding: 5px;
  box-shadow: 0 0 10px #ccc;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%); // 再向左和向上移动相对这个盒子的50%
  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #eee;
  }
}
// 表单
.login_form {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translate(-50%);
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
</style>
