<template>
    <el-container style="height: 100%">
        <el-header>
            <h1>Neusoft&nbsp;&nbsp;东软体检报告管理系统</h1>
            <p>医生：{{ doctor.realName }}</p>
        </el-header>
        <el-container>
            <el-aside width="260px">
                <el-descriptions class="margin-top" title="预约客户信息" :column="1" border>
                    <el-descriptions-item>
                        <template #label>
                            <div class="cell-item">预约编号</div>
                        </template>
                        {{ orders.orderId }}
                    </el-descriptions-item>
                    <el-descriptions-item>
                        <template #label>
                            <div class="cell-item">手机号码</div>
                        </template>
                        {{ orders.userId }}
                    </el-descriptions-item>
                    <el-descriptions-item>
                        <template #label>
                            <div class="cell-item">真实姓名</div>
                        </template>
                        {{ orders.users.realName }}
                    </el-descriptions-item>
                    <el-descriptions-item>
                        <template #label>
                            <div class="cell-item">性别</div>
                        </template>
                        {{ orders.users.sex === 1 ? "男" : "女"}}
                    </el-descriptions-item>
                    <el-descriptions-item>
                        <template #label>
                            <div class="cell-item">套餐类型</div>
                        </template>
                        {{ orders.setMeal.name }}
                    </el-descriptions-item>
                    <el-descriptions-item>
                        <template #label>
                            <div class="cell-item">体检日期</div>
                        </template>
                        {{ orders.orderDate }}
                    </el-descriptions-item>
                </el-descriptions>
                <el-button type="primary" style="margin-top: 20px">查询体检用户</el-button>
            </el-aside>
            <el-main>
                <div class="main-box">
                    <el-collapse>
                        <el-collapse-item v-for="cir in ciReportForm" :key="cir.ciId" :title="cir.ciName">
                            <el-row style="color: #888">
                                <el-col v-for="cidr in cir.cidrList" :key="cidr.cidrId">
                                    <span style="background-color: #ba634e;color: #fff;
                                            box-sizing: border-box;
                                            padding: 1px 3px;
                                            border-radius: 3px;
                                            margin-right: 5px;">
                                        异
                                    </span>
                                    <span style="margin-right: 5px; vertical-align: top">
                                        {{ cidr.name }}
                                    </span>
                                    <el-input style="width: 26%; margin-right: 2px" size="small"/>
<!--                                    <el-input style="width: 80%" type="textarea"/>-->
                                    <span style="margin-right: 15px">{{  }}</span>
                                    <span>正常值范围: {{ cidr.minRange }} - {{ cidr.maxRange }}</span>
                                </el-col>
                            </el-row>
                            <el-button type="primary" style="margin-top: 8px">
                                {{ cir.ciName }}项保存
                            </el-button>
                        </el-collapse-item>
                    </el-collapse>
                    <el-card class="box-card" style="margin-top: 20px">
                        <template #header>
                            <div class="card-header">
                                <span>总检结论</span>
                                <el-button class="button" type="danger">
                                    体检套餐总检结果报告归档
                                </el-button>
                            </div>
                        </template>
                        <div>
                            <el-table style="width: 100%">
                                <el-table-column prop="code" label="编号" width="60" />
                                <el-table-column prop="title" label="总检结论项标题" width="180"/>
                                <el-table-column prop="content" label="总检结论项内容" />
                                <el-table-column label="操作" width="120">
                                    <template #default="">
                                        <el-button type="text" size="small">
                                            更新
                                        </el-button>
                                        <el-button type="text" size="small">
                                            删除
                                        </el-button>
                                    </template>
                                </el-table-column>
                            </el-table>
                            <el-form ref="formRef" style="margin-top: 20px" label-width="120px">
                                <el-form-item label="总检结论标题">
                                    <el-input placeholder="总检结论标题"/>
                                </el-form-item>
                                <el-form-item label="总检结论内容">
                                    <el-input :rows="2" type="textarea" placeholder="总检结论内容"/>
                                </el-form-item>
                                <el-form-item>
                                    <el-button type="primary">
                                        添加
                                    </el-button>
                                    <el-button type="warning">
                                        清空
                                    </el-button>
                                </el-form-item>
                            </el-form>
                        </div>
                    </el-card>

                    <!-- 总检结论更新用对话框 -->
                    <el-dialog title="总检结论项更新" width="60%">
                        <span>
                            <el-form  label-width="120px">
                                <el-form-item label="总检结论标题">
                                    <el-input ></el-input>
                                </el-form-item>
                                <el-form-item label="总检结论内容">
                                    <el-input type="textarea" :rows="3"></el-input>
                                </el-form-item>
                                <el-form-item>
                                    <el-button type="primary">更新</el-button>
                                    <el-button>取消</el-button>
                                </el-form-item>
                            </el-form>
                        </span>
                    </el-dialog>
                </div>
            </el-main>
        </el-container>
    </el-container>
</template>

<script>
import { reactive, toRefs } from "vue";
import { useRouter, useRoute } from "vue-router";
import { getSessionStorage } from "@/common.js";
import axios from "axios";
axios.defaults.baseURL = "http://localhost:8099/tijiancms/";

export default {
    setup() {
        const router = useRouter();
        const route = useRoute();

        const state = reactive({
            doctor: getSessionStorage("doctor"),
            orderId: route.query.orderId,
            orders: {
                setMeal:{},
                users:{}
            },
            ciReportForm: []

        });

        function getOrdersById() {
            axios.post("orders/getOrdersById",{
                orderId:state.orderId
            }).then((response) => {
                // console.log(response.data);
                state.orders = response.data;
            }).catch((error) => {
                console.error(error);
            })
        }

        function listCiReport(orderId) {
            axios.post("ciReport/listCiReport", {
                orderId:orderId
            }).then((response) => {
                // console.log(response.data);
                state.ciReportForm = response.data;
            }).catch((error) => {
                console.error(error);
            })
        }

        getOrdersById();
        listCiReport(state.orderId);

        return {
            ...toRefs(state),
        };

    },
};
</script>

<style scoped>
.el-header {
  background-color: #b3c0d1;
  color: var(--el-text-color-primary);
  text-align: center;
  line-height: 60px;

  display: flex;
  align-items: center;
  justify-content: space-between;
  color: #1c51a3;
}

.el-header h1 {
  font-size: 22px;
  font-weight: 700;
}
.el-header p {
  font-size: 16px;
}

.el-aside {
  background-color: #d3dce6;
  box-sizing: border-box;
  padding: 20px;
}

.el-main {
  background-color: #e9eef3;
  padding: 0;
}

.el-main .main-box {
  width: 100%;
  height: 680px;
  overflow: auto;
  background-color: #fff;
  box-sizing: border-box;
  padding: 20px;
}

/*********** 描述列表 ***********/

.el-descriptions {
  margin-top: 20px;
}
.cell-item {
  display: flex;
  align-items: center;
}
.margin-top {
  margin-top: 20px;
}

/*********** 总检结论 ***********/
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.box-card {
  width: 100%;
}
</style>