<template>
  <div class="login_container">
    <div class="login_box">
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="头像" />
      </div>
      <el-form class="login_form" :model="form" :rules="rules" ref="loginform">
        <el-form-item prop="name">
          <el-input
            prefix-icon="iconfont icon-user"
            v-model="form.name"
          ></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            v-model="form.password"
            type="password"
          ></el-input>
        </el-form-item>
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetForm('loginform')"
            >重置</el-button
          >
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      form: {
        name: 'admin',
        password: '123456'
      },
      rules: {
        name: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 2,
            max: 10,
            message: '用户名长度在5~10个字符',
            trigger: 'blur'
          }
        ],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
      }
    }
  },
  methods: {
    submitForm (formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          alert('submit!')
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    resetForm (formName) {
      console.log(this.$refs[formName])
      this.$refs[formName].resetFields()
    },
    login () {
      this.$refs.loginform.validate(async valid => {
        // 如果检验成功则发出登录请求
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post('login', {
          username: this.form.name,
          password: this.form.password
        })
        if (res.meta.status !== 200) {
          return this.$message.error('登录失败')
        }
        this.$message.success('登录成功')
        // 将token保存至sessionStorage中
        window.sessionStorage.setItem('token', res.data.token)
        // 使用编程式导航跳转到后台主页，路由地址为 '/home'
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
.login_form {
  position: absolute;
  bottom: 0px;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
.avatar_box {
  width: 130px;
  height: 130px;
  border: wheat 1px solid;
  border-radius: 50%;
  padding: 15px;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -35%);
  background-color: #fff;
  box-shadow: 2px 3px wheat;
  img {
    height: 100%;
    width: 100%;
    border-radius: 50%;
    border: wheat 1px solid;
  }
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
