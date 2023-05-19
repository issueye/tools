<template>
    <div class="content">
        <el-card>
            <codemirror 
            v-model:value="code" 
            :options="options" 
            border 
            :height="700" />
        </el-card>
    </div>
</template>

<script setup name="log">
import { reactive, ref } from 'vue';
import Codemirror, { createLog, createTitle } from 'codemirror-editor-vue3';

const code = ref(`
${createTitle('采集输出日志')}
`);
const options = reactive({
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

</script>

<style>
    .content{
    /* width: 900px; */
    text-align: left;
}
</style>