<template>
  <div class="login">
    <!--
      el-form: 整个form组件
      :model： 用于收集整个表单的值  每个表单元素应该和form中的属性双向绑定
      label-width="80px"  设置label的宽度
      el-form-item： 一个表单项

      表单校验
        1. 给el-form传入一个rules规则
        2. 要给el-form-item 添加一个prop属性，值就是字段名
    -->
    <el-form status-icon ref="form" :rules="rules" :model="form" label-width="80px">
      <img src="@/assets/avatar.jpg" alt>
      <el-form-item label="用户名" prop="username">
        <el-input v-model="form.username" placeholder="请输入用户名"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input
          v-model="form.password"
          type="password"
          placeholder="请输入密码"
          @keyup.enter.native="login"
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="login">登录</el-button>
        <el-button @click="reset">重置</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
// 导入axios
// import axios from 'axios'
export default {
  data() {
    return {
      // 收集表单数据
      form: {
        username: '',
        password: ''
      },
      // 表单的校验规则
      rules: {
        // 用户名的校验规则
        username: [
          // 非空校验 trigger： blur  change
          { required: true, message: '用户名不能为空', trigger: 'change' },
          {
            min: 3,
            max: 9,
            message: '用户长度在 3 到 9 个字符',
            trigger: 'change'
          }
        ],
        password: [
          // 非空校验
          { required: true, message: '密码不能为空', trigger: 'change' },
          {
            min: 6,
            max: 12,
            message: '用户长度在 6 到 12 个字符',
            trigger: 'change'
          }
        ]
      }
    }
  },
  methods: {
    reset() {
      // 获取到整个表单组件，重置
      this.$refs.form.resetFields()
    },
    login() {
      // 让整个表单校验
      this.$refs.form.validate(async valid => {
        // valid如果为true，就表示通过 否则不通过
        if (!valid) return false
        // 发送ajax请求，进行登录
        let res = await this.axios({
          method: 'post',
          url: 'login',
          data: this.form
        })
        let {
          meta: { status, msg },
          data: { token }
        } = res
        if (status === 200) {
          // alert('登录成功')
          this.$message.success('登录成功')
          // 把后台颁发的token存起来
          localStorage.setItem('token', token)
          // 跳转到Home组件
          // 参数：跳转的路径
          // 类似于location.href
          this.$router.push('/home')
        } else {
          // 失败的消息  this.$message: 弹出一个消息提示
          this.$message({
            message: msg,
            type: 'error',
            duration: 1000
          })
        }
      })
    }
  }
}
</script>

<style lang="less">
.login {
  width: 100%;
  height: 100%;
  background: #2d434c;
  overflow: hidden;
  .el-form {
    width: 400px;
    margin: 200px auto;
    background-color: #fff;
    padding: 75px 40px 15px;
    border-radius: 10px;
    position: relative;
    img {
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      top: -70px;
      border-radius: 50%;
      border: 10px solid #fff;
    }

    .el-button:nth-child(2) {
      margin-left: 80px;
    }
  }
}
</style>
