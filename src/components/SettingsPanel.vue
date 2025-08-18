<template>
    <div class="settings-panel">
        <!-- 标签全局设置 -->
        <div class="label-settings">
            <h4>画布设置</h4>
            <el-form :model="labelSettings" label-width="60px">
                <el-form-item label="宽">
                    <el-input type="number" v-model="labelSettings.width" @change="updateLabel" />
                </el-form-item>
                <el-form-item label="高">
                    <el-input type="number" v-model="labelSettings.height" @change="updateLabel" />
                </el-form-item>
                <el-button @click="clearCanvas">清空画布</el-button>
                <!-- <el-form-item label="背景">
                    <el-upload :auto-upload="false" :on-change="uploadBackground">
                        <el-button type="primary">选择背景图</el-button>
                    </el-upload>
                </el-form-item> -->
            </el-form>
        </div>

        <!-- 当前元素属性设置（根据元素类型动态显示） -->
        <div v-if="currentElement" class="element-settings">
            <h4>元素设置</h4>
            <component :is="getSettingsComponent(currentElement.type)" :element="currentElement"
                @update-element="updateElement" />
        </div>
        <div v-if="currentElement">

        </div>
    </div>
</template>

<script setup lang="ts">
import { defineProps, defineEmits } from 'vue';
import TextSettings from './TextSettings.vue';
import BarcodeSettings from './BarcodeSettings.vue';
import LineSettings from './LineSettings.vue';
import RectangleSettings from './RectangleSettings.vue';
import TableSettings from './TableSettings.vue'; 


const props = defineProps({ currentElement: Object, labelSettings: Object });
const emits = defineEmits(['update-label', 'update-element', 'clear-canvas']);

// 元素设置组件映射
const settingsComponents = {
    text: TextSettings,
    barcode: BarcodeSettings, 
    qrcode: BarcodeSettings, 
    line: LineSettings,
    vline: LineSettings,
    rectangle: RectangleSettings,
    table: TableSettings  // 添加表格设置组件
};
const getSettingsComponent = (type: string) => settingsComponents[type] || TextSettings;

// 更新标签全局设置
const updateLabel = () => emits('update-label', { ...props.labelSettings });

// 上传背景图（转为Base64）
// const uploadBackground = (file: any) => {
//     const reader = new FileReader();
//     reader.onload = (e) => emits('update-label', { ...props.labelSettings, background: e.target?.result });
//     reader.readAsDataURL(file.raw);
// };
// 清空画布方法
const clearCanvas = () => {
    emits('clear-canvas');
};
// 更新元素属性
const updateElement = (updatedElement: any) => emits('update-element', updatedElement);
</script>

<style scoped>
.settings-panel {
    padding: 10px;
}

.label-settings {
    margin-bottom: 20px;
    padding-bottom: 20px;
    border-bottom: 1px solid #eee;
}

.element-settings {
    margin-top: 20px;
}
</style>