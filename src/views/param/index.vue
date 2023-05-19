<template>
<div class="content">
    <el-card class="form-box">
        <el-form :model="form" label-width="120px">
            <el-form-item label="检索" :inline="true">
                <el-col :span="8">
                    <el-input v-model="form.condition" />
                </el-col>
                <el-col :span="4" style="text-align: center;">
                    <el-button type="primary" size="small" @click="handleQuery">查询</el-button>
                </el-col>
                
            </el-form-item>
        </el-form>
    </el-card>
    <el-card>
        <el-table :data="tableData" stripe style="width: 100%">
            <el-table-column prop="name" label="参数名称" width="200" />
            <el-table-column prop="value" label="参数值" width="150" />
            <el-table-column prop="mark" label="说明" min-width="200" />
            <!-- 操作按钮 -->
            <el-table-column fixed="right" label="操作" width="120">
                <template #default>
                    <el-button type="primary" size="small" @click="handleModify">修改</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
</div>
</template>

<script setup name="param">
import { reactive, ref } from "vue"
import { apiParamList } from '../../api/param.js'

const form = reactive({
    condition: ''
});

// 表格
const tableData = ref([]);

// 获取参数列表
const getData = async () => {
    let { data } = await apiParamList(form);
    console.log('data', data);
    if (data.code == 200) {
        tableData.value = data.data
    }
}

getData()

// 修改参数
const handleModify = () => {
    console.log('123123')
}

const handleQuery = () => {
    console.log('handleQuery');
    getData()
}

</script>

<style>

.form-box {
    margin-bottom: 10px;
}

.content {
    /* width: 900px; */
    text-align: left;
}
</style>
