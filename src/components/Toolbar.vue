<template>
    <div class="toolbar">
        <div class="tool-group">
            <h4>文本</h4>
            <el-button @click="selectTool('text')">
                <el-icon>
                    <Plus />
                </el-icon>
                添加文本</el-button>
        </div>
        <div class="tool-group">
            <h4>多行文本</h4>
            <el-button @click="selectTool('textarea')">
                <el-icon>
                    <Plus />
                </el-icon>
                添加多行文本
            </el-button>
        </div>
        <div class="tool-group">
            <h4>横线</h4>
            <el-button @click="selectTool('line')">
                <el-icon>
                    <Plus />
                </el-icon>
                添加横线</el-button>
        </div>
        <div class="tool-group">
            <h4>竖线</h4>
            <el-button @click="selectTool('vline')">
                <el-icon>
                    <Plus />
                </el-icon>
                添加竖线</el-button>
        </div>
        <div class="tool-group">
            <h4>条码</h4>
            <el-button @click="selectTool('barcode')">
                <el-icon>
                    <Plus />
                </el-icon>
                添加条码</el-button>
        </div>
        <div class="tool-group">
            <h4>二维码</h4>
            <el-button @click="selectTool('qrcode')">
                <el-icon>
                    <Plus />
                </el-icon>
                添加二维码
            </el-button>
        </div>
        <div class="tool-group">
            <h4>表格</h4>
            <el-button @click="selectTool('table')">
                <el-icon>
                    <Plus />
                </el-icon>
                添加表格
            </el-button>
        </div>
        <!-- <div class="tool-group">
            <h4>矩形</h4>
            <el-button icon="Menu" @click="selectTool('rectangle')">添加矩形</el-button>
        </div> -->
        <!-- 弹窗用于配置表格 -->
        <el-dialog v-model="tableDialogVisible" title="表格配置" width="500px">
            <el-form :model="tableConfig" label-width="100px">
                <el-form-item label="行数">
                    <el-input-number v-model="tableConfig.rows" :min="1" :max="20" />
                </el-form-item>
                <el-form-item label="列数">
                    <el-input-number v-model="tableConfig.cols" :min="1" :max="20" />
                </el-form-item>
                <el-form-item label="表格宽度">
                    <el-input-number v-model="tableConfig.width" :min="50" :max="1000" /> px
                </el-form-item>
                <el-form-item label="表格高度">
                    <el-input-number v-model="tableConfig.height" :min="50" :max="1000" /> px
                </el-form-item>
                <!-- <el-form-item label="行高">
                    <el-input-number v-model="tableConfig.rowHeight" :min="10" :max="200" /> px
                </el-form-item>
                <el-form-item label="列宽">
                    <el-input-number v-model="tableConfig.colWidth" :min="10" :max="200" /> px
                </el-form-item> -->
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="tableDialogVisible = false">取消</el-button>
                    <el-button type="primary" @click="confirmTable">确定</el-button>
                </span>
            </template>
        </el-dialog>
    </div>
</template>

<script setup lang="ts">
import { ref, reactive,defineEmits } from 'vue';
import {  Plus } from '@element-plus/icons-vue';

const emits = defineEmits(['select-tool']);
const selectTool = (tool: string) => {
    if (tool === 'table') {
        tableDialogVisible.value = true;
    } else {
        emits('select-tool', tool)
    }
};
// 表格配置弹窗相关
const tableDialogVisible = ref(false);
const tableConfig = reactive({
    rows: 3,
    cols: 3,
    width: 300,
    height: 150,
    rowHeight: 50,
    colWidth: 100
});

const confirmTable = () => {
    emits('select-tool', 'table', tableConfig);
    tableDialogVisible.value = false;
};
</script>

<style scoped>
.toolbar {
    width: 200px;
    padding: 10px;
    border-right: 1px solid #eee;
}

.tool-group {
    margin-bottom: 20px;
}

.tool-group h4 {
    font-size: 14px;
    margin-bottom: 10px;
}
</style>