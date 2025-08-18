<template>
    <el-form label-width="80px">
        <!-- 边框颜色 -->
        <el-form-item label="边框颜色">
            <el-color-picker v-model="form.borderColor" @change="updateStyle" />
        </el-form-item>

        <!-- 填充颜色 -->
        <el-form-item label="填充颜色">
            <el-color-picker v-model="form.fillColor" @change="updateStyle" />
        </el-form-item>

        <!-- 边框粗细 -->
        <el-form-item label="边框粗细">
            <el-input-number v-model="form.borderWidth" :min="1" :max="10" @change="updateStyle" />
        </el-form-item>

        <!-- 宽度 -->
        <el-form-item label="宽度">
            <el-input-number v-model="form.width" :min="1" @change="updateSize" />
        </el-form-item>

        <!-- 高度 -->
        <el-form-item label="高度">
            <el-input-number v-model="form.height" :min="1" @change="updateSize" />
        </el-form-item>
    </el-form>
</template>

<script setup lang="ts">
import { reactive, defineProps, defineEmits, watch } from 'vue';

const props = defineProps({ element: Object });
const emits = defineEmits(['update-element']);

// 表单状态
const form = reactive({
    borderColor: props.element.style?.border?.split(' ')[1] || '#000',
    fillColor: props.element.style?.background || 'transparent',
    borderWidth: parseInt(props.element.style?.border?.split(' ')[0]) || 1,
    width: props.element.size?.width || 100,
    height: props.element.size?.height || 50
});

// 监听外部传入的 element 变化
watch(() => props.element, (newVal) => {
    form.borderColor = newVal.style?.border?.split(' ')[1] || '#000';
    form.fillColor = newVal.style?.background || 'transparent';
    form.borderWidth = parseInt(newVal.style?.border?.split(' ')[0]) || 1;
    form.width = newVal.size?.width || 100;
    form.height = newVal.size?.height || 50;
}, { deep: true });

// 更新样式（边框、背景）
const updateStyle = () => {
    emits('update-element', {
        ...props.element,
        style: {
            ...props.element.style,
            border: `${form.borderWidth}px solid ${form.borderColor}`,
            background: form.fillColor
        }
    });
};

// 更新尺寸（宽高）
const updateSize = () => {
    emits('update-element', {
        ...props.element,
        size: {
            width: form.width,
            height: form.height
        }
    });
};
</script>