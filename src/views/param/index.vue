<template>
<div class="content">
    <el-card class="form-box">
        <el-form :model="form" label-width="120px">
            <el-form-item label="检索" :inline="true">
                <el-col :span="8">
                    <el-input v-model="form.condition" />
                </el-col>
                <el-col :span="4">
                    <span class="form-button">
                        <el-button type="primary" @click="handleQuery">查询</el-button>
                    </span>
                </el-col>
                
            </el-form-item>
        </el-form>
    </el-card>
    <el-card>
        <el-table 
            :data="tableData" 
            stripe 
            border 
            style="width: 100%"
        >
            <el-table-column prop="id" label="编码" width="180" />
            <el-table-column prop="name" label="参数名称" width="200" />
            <el-table-column prop="value" label="参数值" width="400" />
            <el-table-column prop="mark" label="说明" min-width="200" />
            <!-- 操作按钮 -->
            <el-table-column fixed="right" label="操作" width="120" align="center">
                <template #default="scope">
                    <el-button type="primary" size="small" @click="handleModify(scope.row)">修改</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
</div>

<!-- 添加、修改弹窗 -->
<el-dialog 
    v-model="dialogVisible" 
    :title="dialogTitle" 
    width="30%" 
    top="1vh" 
    :close-on-click-modal="false" 
    :before-close="handleDiagClose"
>
    <el-form :model="editForm" :label-position="right" label-width="80px">
        <el-form-item label="名称">
            <el-input v-model="editForm.name" />
        </el-form-item>
        <el-form-item label="参数值">
            <el-input v-model="editForm.value" />
        </el-form-item>
        <el-form-item label="说明">
            <el-input v-model="editForm.mark" />
        </el-form-item>
    </el-form>
    <template #footer>
        <span class="dialog-footer">
            <el-button @click="dialogVisible = false">关闭</el-button>
            <el-button type="primary" @click="handelSave">确定</el-button>
        </span>
    </template>
</el-dialog>
</template>

<script setup name="param">
import { reactive, ref } from "vue"
import { apiParamList, apiParamModify } from '../../api/param.js'
import { ElMessage } from 'element-plus'

const form = reactive({
    condition: ''
});

const editForm = reactive({
    id: '',
    name: '',
    value: '',
    mark: '',
});

// 弹窗
const dialogVisible = ref(false);
const dialogTitle = ref('修改');

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
const handleModify = (row) => {
    dialogVisible.value = true
    editForm.id = row.id
    editForm.name = row.name
    editForm.value = row.value
    editForm.mark = row.mark
}

// 保存修改
const handelSave = async() => {
    let { data } = await apiParamModify(editForm)
    if (data.code == 200) {
        ElMessage({
            type: 'success',
            message: data.message,
        })
    } else {
        ElMessage({
            type: 'error',
            message: data.message,
        })
    }

    dialogVisible.value = false
    getData()
}

const handleQuery = () => {
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
