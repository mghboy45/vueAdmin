<template>
  <div class='login_container'>
   <div class='login_box'>
     <div class='avatar_box'>
       <img src="../assets/logo.png" alt="">
     </div>
      <el-form ref='loginFromRef' :rules='loginFromRules' :model="loginFrom" class='login_from'>
        <el-form-item prop='username'>
          <el-input v-model="loginFrom.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <el-form-item prop='password'>
          <el-input type='password' v-model="loginFrom.password" prefix-icon="iconfont icon-3702mima"></el-input>
        </el-form-item>
        <el-form-item class='btns'>
          <el-button type='primary' @click='login'>登录</el-button>
          <el-button type='info' @click='resetLoginFrom'>重置</el-button>
        </el-form-item>
      </el-form>
   </div>
  </div>
</template>

<script>
export default {
  name: 'login',
  data () {
    return {
      loginFrom: {
        username: 'admin',
        password: '123456'
      },
      loginFromRules: {
        username: [
          {
            required: true, message: '请输入用户名', trigger: 'blur'
          }
        ],
        passwrod: [
          {
            required: true, message: '请输入密码', trigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    resetLoginFrom () {
      this.$refs.loginFromRef.resetFields()
    },
    login () {
      this.$refs.loginFromRef.validate(async bool => {
        if (!bool) return
        const { data: res } = await this.$http.post('login', this.loginFrom)
        if (res.meta.status !== 200) {
          this.$message.error('登录失败')
        }
        this.$message.success('登录成功')
        // 将登录成功之后的 token 存入 sessionStorage
        window.sessionStorage.setItem('token', res.data.token)
        // 保存 token 之后进行页面跳转
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}
.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.avatar_box {
  height: 130px;
  width: 130px;
  border: 1px solid #eee;
  border-radius: 50%;
  padding: 10px;
  box-shadow: 0 0 10px #ddd;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
}
.avatar_box img {
  width:100%;
  height: 100%;
  border-radius: 50%;
  background-color: #eee;
}
.login_from {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 20px;
  box-sizing: border-box;
}

.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
