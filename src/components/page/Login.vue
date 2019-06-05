<template>
  <div class="login-wrap">
    <div class="ms-login">
      <div class="ms-title">XXX后台管理系统</div>
      <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="0px" class="ms-content">
        <el-form-item prop="username">
          <el-input v-model="ruleForm.username" placeholder="username">
            <el-button slot="prepend" icon="el-icon-lx-people"></el-button>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            type="password"
            placeholder="password"
            v-model="ruleForm.password"
            @keyup.enter.native="submitForm('ruleForm')"
          >
            <el-button slot="prepend" icon="el-icon-lx-lock"></el-button>
          </el-input>
        </el-form-item>
        <div class="login-btn">
          <el-button type="primary" @click="submitForm('ruleForm')">登录</el-button>
        </div>
        <p class="login-tips">Tips : 用户名和密码随便填。</p>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data: function() {
    return {
      ruleForm: {
        username: "admin",
        password: "1231231"
      },
      rules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" }
        ],
        password: [{ required: true, message: "请输入密码", trigger: "blur" }]
      }
    };
  },
  methods: {
    submitForm(formName) {
      // // 使用vue-resource 进行数据调用
      //  let formData = new FormData();
      // formData.append("uname",this.ruleForm.username);
      // formData.append("upwd",this.ruleForm.password);
      // const url = "http://localhost:8086/user/login"
      // var data = {
      //     "uname":this.ruleForm.username,
      //     "upwd":this.ruleForm.password
      // }
      // console.log(data);
      // this.$http.post(url,formData).then(
      //     (response) =>{
      //         console.log(response);
      //     },(response)=>{
      //         console.log(+response);
      //     })
      // // 使用vue-resource 进行数据调用 end

      this.$refs[formName].validate(valid => {
        if (valid) {
          this.$ajax
            .post(
              "http://localhost:8086/user/login",
              this.$qs.stringify({
                uname: this.ruleForm.username,
                upwd: this.ruleForm.password
              })
            )
            .then(response => {
              console.log(response);
              if (response.data == "") {
                alert("username or password error");
                this.ruleForm.password='';
                this.$router.push("/login");
              } else {
                const username = response.data.xUsername;
                localStorage.setItem("ms_username", username);
                this.$router.push("/");
              }
            })
            .catch(function(response) {
              alert("系统异常，请联系管理员");
              this.$router.push("/404");
            });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    }
  }
};
</script>

<style scoped>
.login-wrap {
  position: relative;
  width: 100%;
  height: 100%;
  background-image: url(../../assets/img/login-bg.jpg);
  background-size: 100%;
}
.ms-title {
  width: 100%;
  line-height: 50px;
  text-align: center;
  font-size: 20px;
  color: palegreen;
  border-bottom: 1px solid #ddd;
}
.ms-login {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 350px;
  margin: -190px 0 0 -175px;
  border-radius: 5px;
  background: rgba(255, 255, 255, 0.3);
  overflow: hidden;
}
.ms-content {
  padding: 30px 30px;
}
.login-btn {
  text-align: center;
}
.login-btn button {
  width: 100%;
  height: 36px;
  margin-bottom: 10px;
}
.login-tips {
  font-size: 12px;
  line-height: 30px;
  color: #fff;
}
</style>