<template>
    <el-container style="height: 100%">

        <el-header>
            <h1>Neusoft&nbsp;&nbsp;东软体检报告管理系统</h1>
            <p>医生</p>
        </el-header>

        <el-container>
            <el-aside width="260px">
                <h4>体检客户查询</h4>
                <el-form  :model="asideForm" label-width="auto">
                    <el-form-item label="手机号码">
                        <el-input v-model="asideForm.userId" placeholder="手机号码"/>
                    </el-form-item>
                    <el-form-item label="姓名">
                        <el-input v-model="asideForm.realName" placeholder="姓名"/>
                    </el-form-item>
                    <el-form-item label="性别">
                        <el-radio-group v-model="asideForm.sex">
                            <el-radio label="1">男</el-radio>
                            <el-radio label="2">女</el-radio>
                        </el-radio-group>
                    </el-form-item>
                    <el-form-item label="套餐类型">
                        <el-select v-model="asideForm.smId" placeholder="套餐类型">
<!--                            <el-option-->
<!--                                v-for="item in options"-->
<!--                                :key="item.key"-->
<!--                                :label="item.label"-->
<!--                                :value="item.value"-->
<!--                                :disabled="item.disabled"-->
<!--                            />-->
                            <el-option v-for="setMeal in setMealArr"
                                       :key="setMeal.smId"
                                       :value="setMeal.smId"
                                       :label="setMeal.name"
                            >

                            </el-option>
<!--                            <el-option label="Zone two" value="Beijing"/>-->
                        </el-select>
                    </el-form-item>
                    <el-form-item label="体检日期">
                        <el-date-picker
                            v-model="asideForm.orderDate"
                            type="date"
                            placeholder="体检日期"
                            format="YYYY/MM/DD"
                            value-format="YYYY-MM-DD"
                        />
                    </el-form-item>
                    <el-form-item label="是否归档">
                        <el-radio-group v-model="asideForm.state">
                            <el-radio label="1">已归档</el-radio>
                            <el-radio label="2">未归档</el-radio>
                        </el-radio-group>
                    </el-form-item>
                </el-form>
                <el-button type="primary" @click="doSelect">查询</el-button>
                <el-button type="warning" @click="reset">重置</el-button>
            </el-aside>
            <el-main>
                <el-scrollbar>
                    <el-table :data="mainTableElement.rowList" stripe style="width: 100%">
                        <el-table-column prop="orderId" label="预约编号" width="180" />
                        <el-table-column prop="userId" label="手机号码" width="180" />
                        <el-table-column prop="users.realName" label="真实姓名" width="180" />
                        <el-table-column prop="users.sex" label="性别" width="60">
                            <template #default="scope">
                                <span>{{ scope.row.users.sex === 1 ? "男" : "女" }}</span>
                            </template>
                        </el-table-column>
                        <el-table-column prop="setMeal.name" label="体检套餐类型" width="180" />
                        <el-table-column prop="hospital.name" label="体检医院" width="200" />
                        <el-table-column prop="orderDate" label="体检日期" width="200" />
                        <el-table-column prop="address" label="操作" width="160">
                            <template #default="scope">
                                <el-button type="text" size="small" @click="ciReport(scope.row)">
                                    编辑体检报告
                                </el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-scrollbar>
                <el-pagination background layout="prev, pager, next, total"
                               :total="mainTableElement.totalRowNum"
                               :page-size="mainTableElement.rowPerPageNum"
                               @current-change="handleCurrPageChange"
                />
            </el-main>
        </el-container>
    </el-container>
</template>

<script>

import axios from "axios";
import {reactive, ref, toRefs} from "vue";
import {getSessionStorage} from "@/common.js";
import {useRouter} from "vue-router";
axios.defaults.baseURL = "http://localhost:8099/tijiancms/";

export default {
    setup() {
        const router = useRouter();

        const state = reactive({
            doctor: getSessionStorage("doctor"),
            asideForm: {
                userId:"",
                realName:"",
                sex:"",
                smId:"",
                orderDate:"",
                state:"1"
            },
            setMealArr: [],
            mainTableElement: {}
        });
        function listSetMeal() {
            axios.post("setMeal/listSetMeal")
                .then((response) => {
                    state.setMealArr = response.data;
                })
                .catch((error) => {
                    console.error(error);
                })
        }
        function listOrdersTable(pageNum) {
            state.asideForm.currPageNum = pageNum;
            state.asideForm.rowPerPageNum = 10;
            axios.post("orders/queryOrdersMultiConditions", state.asideForm)
                .then((response) => {
                    // console.log(response.data);
                    state.mainTableElement = response.data;
                })
                .catch((error) => {
                    console.error(error);
                })
        }
        function handleCurrPageChange(pageNum) {
            // console.log(pageNum);
            listOrdersTable(pageNum);
        }
        function doSelect() {
            listOrdersTable(1);
        }
        function reset() {
            state.asideForm = {
                userId:"",
                realName:"",
                sex:"",
                smId:"",
                orderDate:"",
                state:"1"
            }
            doSelect();
        }
        function ciReport(row) {
            //向后台请求调用创建体检报告模板的方法
            axios.post("ciReport/createReportTemplate", {
                orderId:row.orderId,
                smId:row.smId
            }).then((response) => {
                if(response.data === 1) {
                    //跳转
                    router.push({path:"ordersContent",query:{orderId:row.orderId}});
                } else {
                    alert("生成报告模板失败");
                }
            }).catch((error) => {
                console.error(error);
            })
        }
        listSetMeal();
        listOrdersTable(1);
        return {
            ...toRefs(state),
            listOrdersTable,
            handleCurrPageChange,
            doSelect,
            reset,
            ciReport
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
.el-aside h4 {
  color: #555;
  margin-bottom: 20px;
}

.el-main {
  background-color: #e9eef3;
}
</style>