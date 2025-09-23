<template>
    <div class="toolbar">
        <div class="tool-group">
            <div class="tool-item" @click="selectTool('text')">
                <el-icon class="tool-icon">
                    <svg xmlns="http://www.w3.org/2000/svg" width="256" height="256" viewBox="0 0 24 24">
                        <path fill="currentColor"
                            d="M5 6a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2v1a1 1 0 1 1-2 0V6h-4v12h1a1 1 0 1 1 0 2h-4a1 1 0 1 1 0-2h1V6H7v1a1 1 0 0 1-2 0z">
                        </path>
                    </svg>
                </el-icon>
                <span class="tool-text">单行文本</span>
            </div>
        </div>
        <div class="tool-group">
            <div class="tool-item" @click="selectTool('textarea')">
                <el-icon class="tool-icon">
                    <svg xmlns="http://www.w3.org/2000/svg" width="256" height="256" viewBox="0 0 24 24">
                        <path fill="currentColor"
                            d="M7.885 19.116v-13h-5V4.885h11.23v1.23h-5v13zm9 0v-8h-3V9.885h7.23v1.23h-3v8z"></path>
                    </svg>
                </el-icon>
                <span class="tool-text">多行文本</span>
            </div>
        </div>

        <div class="tool-group">
            <div class="tool-item" @click="selectTool('line')">
                <el-icon class="tool-icon">
                    <svg xmlns="http://www.w3.org/2000/svg" width="256" height="256" viewBox="0 0 15 15">
                        <path fill="currentColor" fill-rule="evenodd"
                            d="M1.25 7.5a.5.5 0 0 1 .5-.5h11.5a.5.5 0 0 1 0 1H1.75a.5.5 0 0 1-.5-.5"
                            clip-rule="evenodd"></path>
                    </svg>
                </el-icon>
                <span class="tool-text">横线</span>
            </div>
        </div>
        <div class="tool-group">
            <div class="tool-item" @click="selectTool('vline')">
                <el-icon class="tool-icon">
                    <svg xmlns="http://www.w3.org/2000/svg" width="256" height="256" viewBox="0 0 20 20">
                        <path fill="currentColor" fill-rule="evenodd"
                            d="M10 .5a.5.5 0 0 0-.5.5v18a.5.5 0 0 0 1 0V1a.5.5 0 0 0-.5-.5" clip-rule="evenodd"></path>
                    </svg>
                </el-icon>
                <span class="tool-text">竖线</span>
            </div>
        </div>
        <div class="tool-group">
            <div class="tool-item" @click="selectTool('rectangle')">
                <el-icon class="tool-icon">
                    <svg xmlns="http://www.w3.org/2000/svg" width="256" height="256" viewBox="0 0 24 24">
                        <path fill="currentColor" d="M2 2h20v20H2zm2 2v16h16V4z"></path>
                    </svg>
                </el-icon>
                <span class="tool-text">矩形</span>
            </div>
        </div>
        <div class="tool-group">
            <div class="tool-item" @click="selectTool('barcode')">
                <el-icon class="tool-icon">
                    <svg xmlns="http://www.w3.org/2000/svg" width="256" height="240.94117647059" viewBox="0 0 17 16">
                        <path fill="currentColor" fill-rule="evenodd"
                            d="M14 2h1.953v10.914H14zm-3 0h1.967v10.914H11zM9 2h.95v10.914H9zM5 2h1.972v10.914H5zM3 2h.973v10.914H3zM1 2h.973v10.914H1z">
                        </path>
                    </svg>
                </el-icon>
                <span class="tool-text">条码</span>
            </div>
        </div>
        <div class="tool-group">
            <div class="tool-item" @click="selectTool('qrcode')">
                <el-icon class="tool-icon">
                    <svg xmlns="http://www.w3.org/2000/svg" width="256" height="256" viewBox="0 0 16 16">
                        <g fill="currentColor">
                            <path d="M2 2h2v2H2z"></path>
                            <path d="M6 0v6H0V0zM5 1H1v4h4zM4 12H2v2h2z"></path>
                            <path d="M6 10v6H0v-6zm-5 1v4h4v-4zm11-9h2v2h-2z"></path>
                            <path
                                d="M10 0v6h6V0zm5 1v4h-4V1zM8 1V0h1v2H8v2H7V1zm0 5V4h1v2zM6 8V7h1V6h1v2h1V7h5v1h-4v1H7V8zm0 0v1H2V8H1v1H0V7h3v1zm10 1h-1V7h1zm-1 0h-1v2h2v-1h-1zm-4 0h2v1h-1v1h-1zm2 3v-1h-1v1h-1v1H9v1h3v-2zm0 0h3v1h-2v1h-1zm-4-1v1h1v-2H7v1z">
                            </path>
                            <path d="M7 12h1v3h4v1H7zm9 2v2h-3v-1h2v-1z"></path>
                        </g>
                    </svg> 
                </el-icon>
                <span class="tool-text">二维码</span>
            </div>
        </div>
        <div class="tool-group">
            <div class="tool-item" @click="selectTool('table')">
                <el-icon class="tool-icon">
                    <svg xmlns="http://www.w3.org/2000/svg" width="256" height="256" viewBox="0 0 24 24">
                        <path fill="currentColor"
                            d="M21.5 2h-19a.5.5 0 0 0-.5.5v19a.5.5 0 0 0 .5.5h19a.5.5 0 0 0 .5-.5v-19a.5.5 0 0 0-.5-.5m-13 19H3v-5.5h5.5zm0-6.5H3v-5h5.5zm0-6H3V3h5.5zm6 12.5h-5v-5.5h5zm0-6.5h-5v-5h5zm0-6h-5V3h5zM21 21h-5.5v-5.5H21zm0-6.5h-5.5v-5H21zm0-6h-5.5V3H21z">
                        </path>
                    </svg>
                </el-icon>
                <span class="tool-text">表格</span>
            </div>
        </div>
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
import { ref, reactive, defineEmits } from 'vue';
import { Plus } from '@element-plus/icons-vue';

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
    width: 110px;
    padding: 10px;
    border-right: 1px solid #eee;
}

.tool-group {
    margin-bottom: 20px;
}

.tool-group h4 {
    font-size: 14px;
    margin-bottom: 10px;
    color: #666;
}

.tool-item {
    display: flex;
    align-items: center;
    padding: 8px;
    cursor: pointer;
    border-radius: 4px;
    transition: background-color 0.2s;
}

.tool-item:hover {
    background-color: #f5f5f5;
}

.tool-icon {
    margin-right: 8px;
    font-size: 20px;
    color: #000000;
}

.tool-text {
    font-size: 14px;
    color: #333;
}
</style>