<template>
    <el-form class="barcode-settings" :model="form" label-width="80px" size="small">
        <!-- 1. 条码基础属性 -->

        <el-form-item label="编码格式">
            <el-select v-model="form.codeType" @change="updateElement" disabled>
                <el-option label="CODE128（通用）" value="CODE128" />
            </el-select>
        </el-form-item>
        <el-form-item label="条码内容">
            <el-input v-model="form.content" placeholder="请输入条码内容（如：123456789012）" @input="updateElement" />
        </el-form-item>

        <!-- <el-form-item label="条码高度">
            <el-input-number v-model="form.height" :min="30" :max="200" suffix="px" @change="updateElement" />
        </el-form-item>

        <el-form-item label="条码边距">
            <el-input-number v-model="form.margin" :min="0" :max="50" suffix="px" @change="updateElement" />
        </el-form-item>

        <el-form-item label="显示文本">
            <el-switch v-model="form.displayText" active-text="显示" inactive-text="隐藏" @change="updateElement" />
        </el-form-item>

        <el-form-item label="文字边距" v-if="form.displayText">
            <el-input-number v-model="form.textMargin" :min="0" :max="30" suffix="px" @change="updateElement" />
        </el-form-item>

        <el-form-item label="文字大小" v-if="form.displayText">
            <el-input-number v-model="form.textSize" :min="10" :max="30" suffix="px" @change="updateElement" />
        </el-form-item>

        <el-form-item label="文字位置" v-if="form.displayText">
            <el-radio-group v-model="form.textPosition" @change="updateElement">
                <el-radio label="bottom">底部</el-radio>
                <el-radio label="top">顶部</el-radio>
            </el-radio-group>
        </el-form-item> -->
    </el-form>
</template>
<script setup lang="ts">
import { ref, reactive, watch, defineProps, defineEmits } from 'vue';

// 1. 接收父组件传递的当前条码元素（必传）
const props = defineProps({
    element: {
        type: Object,
        required: true,
        default: () => ({})
    }
});

// 2. 触发父组件更新条码元素（传递修改后的element）
const emits = defineEmits(['update-element']);

// 3. 表单状态（双向绑定配置项，初始值从element中获取）
const form = reactive({
    codeType: 'CODE128', // 条码类型（默认通用CODE128）
    content: '', // 条码内容（必填）
    height: 50, // 条码高度（px）
    margin: 10, // 条码边距（px）
    displayText: true, // 是否显示条码下方文本（默认显示）
    textMargin: 2, // 文字边距（px）
    textPosition: 'bottom', // 文本位置（默认底部）
    textSize: 12 // 文本大小（px）
});


// 4. 初始化表单：从当前条码元素中同步初始值
const initForm = () => {
    // 基础属性
    form.type = props.element.type || 'CODE128';
    form.content = props.element.content || '';
    // 样式属性（从element.style中获取，无则用默认值）
    form.height = props.element.style?.height ? parseInt(props.element.style.height) : 50;
    form.margin = props.element.style?.margin ? parseInt(props.element.style.margin) : 10;
    // 文本配置
    form.textMargin = props.element.style?.textMargin ? parseInt(props.element.style?.textMargin) : 2;
    form.displayText = props.element.style?.displayText ?? true;
    form.textPosition = props.element.style?.textPosition || 'bottom';
    form.textSize = props.element.style?.textSize ? parseInt(props.element.style.textSize) : 12;
};

// 5. 监听条码元素变化：当元素从父组件更新时，同步表单状态
watch(() => props.element, () => {
    initForm();
}, { deep: true }); // 深度监听（element.style是对象）

// 6. 组件挂载时初始化表单
initForm();

// 7. 更新条码元素：将表单配置同步到父组件
const updateElement = () => {
    // 构造修改后的条码元素（合并原元素属性与新配置）
    const updatedElement = {
        ...props.element,
        // type: form.type, // 更新条码类型
        content: form.content, // 更新条码内容
        style: {
            ...props.element.style, // 保留原样式
            height: `${form.height}px`, // 高度（带单位）
            margin: `${form.margin}px`, // 边距（带单位）
            displayText: form.displayText, // 是否显示文本
            textMargin: `${form.textMargin}px`, // 文字边距（带单位）
            textPosition: form.textPosition, // 文本位置
            textSize: `${form.textSize}px` // 文本大小（带单位）
        }
    };

    // 触发父组件事件，更新条码元素
    emits('update-element', updatedElement);
};
</script>
<style scoped>
.barcode-settings {
    padding: 10px 0;
}

/* 表单项间距调整 */
.el-form-item {
    margin-bottom: 15px;
}

/* 输入框宽度占满 */
.el-input-number,
.el-color-picker {
    width: 100%;
}

/* 文本位置单选框布局 */
.el-radio-group {
    display: flex;
    gap: 10px;
    align-items: center;
    margin-top: 5px;
}
</style>