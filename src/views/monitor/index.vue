<template>
<div class="content">
    <el-card class="form-box">
        <el-form 
            :model="form" 
            label-width="120px"
        >
            <el-form-item 
                label="检索" 
                :inline="true"
            >
                <el-col :span="4">
                    <el-input v-model="form.condition" />
                </el-col>
                <el-col :span="4">
                    <span class="form-button">
                        <el-button 
                            type="primary" 
                            @click="handleQuery"
                        >查询</el-button>
                    </span>
                    <span class="form-button">
                        <el-button 
                            type="primary" 
                            @click="handleAdd"
                        >添加</el-button>
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
            <el-table-column fixed prop="id" label="编码" width="200" />
            <el-table-column fixed prop="name" label="名称" width="150" />
            <el-table-column prop="logPath" label="日志路径" min-width="300" />
            <el-table-column prop="level" label="日志等级" width="200">
                <template #default="scope">
                    {{ getLevelDisplay(scope.row) }}
                </template>
            </el-table-column>
            <el-table-column prop="scriptPath" label="脚本路径" min-width="300" />
            <el-table-column fixed="right" prop="state" label="状态" width="100" align="center">
                <template #default="scope">
                    <el-tag :type="scope.row.state ? 'success' : 'danger'" disable-transitions>{{ scope.row.state ? '启用'
                            :
                            '停用' }}</el-tag>
                </template>
            </el-table-column>
            <el-table-column prop="createTime" label="创建时间" min-width="300" />
            <!-- 操作按钮 -->
            <el-table-column fixed="right" label="操作" width="300" align="center">
                <template #default="scope">
                    <el-button type="primary" size="small" @click="handleViewLog(scope.row)">查看日志</el-button>
                    <el-button type="primary" size="small" @click="handleEnable(scope.row)">{{ scope.row.state ?
                            '停用' : '启用' }}</el-button>
                    <el-button type="primary" size="small" @click="handleModify(scope.row)">修改</el-button>
                    <el-button type="danger" size="small" @click="handleClose(scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
</div>

<!-- 日志弹窗 -->
<el-dialog 
    v-model="dialogLogViewVisible" 
    title="查看监听日志" 
    width="50%" 
    top="1vh" 
    :close-on-click-modal="false" 
    @open="handleOpenLogView"
    @opened="handleDiagLogViewOpened"
>
    <codemirror 
        v-model:value="code" 
        :options="logOptions" 
        border 
        :height="500" 
        @change="handleCodeChange"/> 
</el-dialog>

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
        <el-form-item label="日志路径">
            <el-input v-model="editForm.logPath" />
        </el-form-item>
        <el-form-item label="日志等级">
            <el-select v-model="editForm.level" placeholder="Select">
                <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
            </el-select>
        </el-form-item>
        <el-form-item label="脚本路径">
            <el-input v-model="editForm.scriptPath" />
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

<script setup name="monitor">
import Codemirror, { createLog, createTitle } from 'codemirror-editor-vue3';
import { reactive, ref } from "vue"
import { apiMonitorList, apiMonitorAdd, apiMonitorModify, apiMonitorModifyState, apiMonitorDel } from '../../api/monitor.js'
import { ElMessage, ElMessageBox } from 'element-plus'

const form = reactive({
    condition: ''
});

const editForm = reactive({
    id: '',
    name: '',
    logPath: '',
    level: '',
    scriptPath: '',
});

const code = ref('');
const logOptions = reactive({
    // autorefresh: true,          // 是否自动刷新
    // smartIndent: true,          // 自动缩进
    tabSize: 4,                     // 缩进单元格为 4 个空格
    mode: 'fclog',                    // 编辑器的编程语言，比如：'javascript'
    line: true,                     // 是否显示行数
    viewportMargin: Infinity,       // 高度自适应
    highlightDifferences: true,
    autofocus: false,
    connect: 'left',
    indentUnit: 2,
    readOnly: true, // 只读
    showCursorWhenSelecting: true,
    firstLineNumber: 1,
    matchBrackets: true,//括号匹配  
});

// 添加编辑弹窗
const dialogVisible = ref(false);
const dialogTitle = ref('添加');
// 日志弹窗是否显示
const dialogLogViewVisible = ref(false);

let ws = null;

const options = [
    { value: -1, label: 'DEBUG'},
    { value: 0, label: 'INFO'},
    { value: 1, label: 'WARN'},
    { value: 2, label: 'ERROR'}
]

// 表格
const tableData = ref([]);

// 获取参数列表
const getData = async () => {
    let {
        data
    } = await apiMonitorList(form);
    console.log('data', data);
    if (data.code == 200) {
        tableData.value = data.data
    }
}

// 代码编辑器发送变化时
const handleCodeChange = (value, cm) => {
    let sc = cm.getScrollInfo();
    cm.scrollTo(sc.left,( sc.height + sc.top));
}

// 打开日志窗口
const handleDiagLogViewOpened = () => {
    console.log('handleDiagLogViewOpened');
    code.value = `${createTitle('采集输出日志')}\n`
}

// 打开日志查看弹窗
const handleOpenLogView = () => {
    console.log('handleOpenLogView', 'handleOpenLogView');


    // 判断是否已经连接，如果已经连接，则关闭连接
    if (ws) {
        ws.close()
    }

    ws = new WebSocket(`ws://${location.host}/ws/monitor/${editForm.id}`)

    ws.onmessage = (event) => {
        console.log('event', event);
        code.value += `${createLog(event.data, 'info')}\n`
    }
}

// 查看日志
const handleViewLog = (row) => {
    dialogLogViewVisible.value = true
    editForm.id = row.id
    editForm.name = row.name
    editForm.logPath = row.logPath
    editForm.level = row.level
    editForm.scriptPath = row.scriptPath
}

// 转换日志等级显示
const getLevelDisplay = (row) => {
    switch (row.level) {
        case -1:
            return "debug"
        case 0:
            return "info"
        case 1:
            return "warn"
        case 2:
            return "error"
    }
}

getData()

// 修改状态
const handleEnable = async (row) => {
    let {
        data
    } = await apiMonitorModifyState(row.id)
    console.log('handleEnable', data);
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

    getData()
}

// 修改参数
const handleModify = (row) => {
    editForm.id = row.id
    editForm.name = row.name
    editForm.logPath = row.logPath
    editForm.level = row.level
    editForm.scriptPath = row.scriptPath

    dialogTitle.value = '修改'
    dialogVisible.value = true
}

const handleClose = (row) => {
    ElMessageBox.confirm(
        '确定是否删除?',
        '删除', {
            distinguishCancelAndClose: true,
            confirmButtonText: '确定',
            cancelButtonText: '取消',
        }
    ).then(async () => {
        // 调用接口
        let {
            data
        } = await apiMonitorDel(row.id)
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
        getData()
    }).catch((action) => {
        ElMessage({
            type: 'info',
            message: action === 'cancel' ?
                '取消删除' :
                'Stay in the current route',
        })
    })
}

const handleQuery = () => {
    console.log('handleQuery');
    getData()
}

// 添加按钮
const handleAdd = () => {
    dialogVisible.value = true;
    dialogTitle.value = '添加';

    editForm.id = ''
    editForm.name = ''
    editForm.logPath = ''
    editForm.level = -1
    editForm.scriptPath = ''
}

// 调用接口
const handelSave = async () => {
    if (dialogTitle.value == '添加') {
        let {
            data
        } = await apiMonitorAdd(editForm)
        if (data.code == 200) {
            ElMessage({
                type: 'success',
                message: data.message,
            })
        }
    }

    if (dialogTitle.value == '修改') {
        let {
            data
        } = await apiMonitorModify(editForm)
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
            return
        }
    }

    dialogVisible.value = false
    getData()
}
</script>

<style>
.form-box {
    margin-bottom: 10px;
}

.form-button {
    margin-left: 10px;
}

.content {
    /* width: 900px; */
    text-align: left;
}
</style>
