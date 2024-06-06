<template>
  <el-card class="box-card">
    <template #header>
      <div class="card-header">
        <span>登录</span>
      </div>
    </template>
    <div class="text item">
        <el-form label-width="120px" :model="loginForm">
            <el-form-item label="医生编码">
                <el-input v-model="loginForm.docCode"/>
            </el-form-item>
            <el-form-item label="登录密码">
                <el-input v-model="loginForm.password" type="password"/>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="login">
                    登录
                </el-button>
            </el-form-item>
        </el-form>
    </div>
  </el-card>
</template>

<script>

import axios from "axios";
import {reactive, toRefs} from "vue";
import {useRouter} from "vue-router";
import {setSessionStorage} from "@/common.js";

axios.defaults.baseURL = "http://localhost:8099/tijiancms/";

export default {
  setup() {
      const router = useRouter();
      //所有相应数据的声明，包括对象、数组
      const state = reactive({
          loginForm: {
              docCode:"",
              password:""
          }
      })

      const login = () => {
          if (state.loginForm.docCode === "") {
              alert("医生编码不能为空");
              return;
          }
          if (state.loginForm.password === "") {
              alert("登录密码不能为空");
              return;
          }
          axios.post("doctor/getDoctorByCodeByPass", state.loginForm)
              .then((response) => {
                  let doctor = response.data;
                  if (doctor !== "") {
                      setSessionStorage("doctor", doctor);
                      router.push("/ordersList");
                  } else {
                      alert("您输入的医生信息有误");
                  }
              })
              .catch((error) => {
                  console.error(error);
              })
      }

      return {
          ...toRefs(state),
          login
      }
  },
};
</script>

<style scoped>
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.text {
  font-size: 14px;
}

.item {
  margin-bottom: 18px;
}

.box-card {
  width: 400px;
  margin: 0 auto;
  margin-top: 150px;
}
</style>