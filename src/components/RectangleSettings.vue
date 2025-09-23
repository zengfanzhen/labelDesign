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
        <!-- 圆角弧度 -->
        <el-form-item label="圆角弧度">
            <el-input-number v-model="form.borderRadius" :min="0" :max="50" @change="updateStyle" />
        </el-form-item>
    </el-form>
</template>

<script setup lang="ts">
import { reactive, defineProps, defineEmits, watch } from 'vue';

const props = defineProps({ element: Object });
const emits = defineEmits(['update-element']);
const rgbToHex = (rgb) => {
    if (!rgb) return '#000000';

    // 如果已经是十六进制格式，直接返回
    if (rgb.startsWith('#')) return rgb;

    // 处理 rgb 格式
    const result = rgb.match(/\d+/g);
    if (!result || result.length < 3) return '#000000';

    const r = parseInt(result[0]).toString(16).padStart(2, '0');
    const g = parseInt(result[1]).toString(16).padStart(2, '0');
    const b = parseInt(result[2]).toString(16).padStart(2, '0');

    return `#${r}${g}${b}`;
};// 改进的颜色提取函数，更好地处理各种颜色格式
const extractBorderColor = (borderStyle) => {
    if (!borderStyle) return '#000';

    // 匹配十六进制颜色 (#xxx 或 #xxxxxx)
    const hexMatch = borderStyle.match(/#([0-9a-fA-F]{3}|[0-9a-fA-F]{6})\b/);
    if (hexMatch) return hexMatch[0];

    // 匹配 RGB/RGBA 颜色并转换为十六进制
    const rgbMatch = borderStyle.match(/rgb\(?([^)]+)\)?/i);
    if (rgbMatch) {
        const values = rgbMatch[1].split(',').map(v => parseInt(v.trim()));
        if (values.length >= 3) {
            const r = values[0].toString(16).padStart(2, '0');
            const g = values[1].toString(16).padStart(2, '0');
            const b = values[2].toString(16).padStart(2, '0');
            return `#${r}${g}${b}`;
        }
    }

    // 匹配颜色名称
    const namedColorMatch = borderStyle.match(/\b[a-zA-Z]+\b/);
    if (namedColorMatch) return namedColorMatch[0];

    return '#000';
};

const form = reactive({
    borderColor: extractBorderColor(props.element.style?.border) || '#000',
    fillColor: props.element.style?.background || 'transparent',
    borderWidth: parseInt(props.element.style?.border?.split(' ')[0]) || 1,
    borderRadius: parseInt(props.element.style?.borderRadius) || 0, // 添加圆角属性
    width: props.element.size?.width || 100,
    height: props.element.size?.height || 50
});

watch(() => props.element, (newVal) => {
    form.borderColor = extractBorderColor(newVal.style?.border) || '#000';
    form.fillColor = newVal.style?.background || 'transparent';
    form.borderWidth = parseInt(newVal.style?.border?.split(' ')[0]) || 1;
    form.borderRadius = parseInt(newVal.style?.borderRadius) || 0; // 同步圆角属性
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
            background: form.fillColor,
            borderRadius: `${form.borderRadius}px` // 添加圆角样式
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