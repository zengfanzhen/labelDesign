<template>
    <el-form label-width="80px">
        <el-form-item label="颜色">
            <el-color-picker v-model="form.color" @change="updateStyle" />
        </el-form-item>
        <el-form-item label="粗细">
            <el-input-number v-model="form.thickness" :min="1" :max="10" @change="updateStyle" />
        </el-form-item>
        <el-form-item label="长度">
            <el-input-number v-model="form.width" :min="1" @change="updateSize" />
        </el-form-item>
        <el-form-item label="旋转角度">
            <el-input-number v-model="form.rotation" :min="0" :max="360" @change="updateStyle" />
        </el-form-item>
        <!-- <el-form-item label="高度">
            <el-input-number v-model="form.height" :min="1" @change="updateSize" />
        </el-form-item> -->
    </el-form>
</template>

<script setup lang="ts">
import { reactive, defineProps, defineEmits, watch } from 'vue';

const props = defineProps({ element: Object });
const emits = defineEmits(['update-element']);

const form = reactive({
    color: props.element.style?.background || '#000', 
    thickness: props.element.size?.height || 3, 
    width: props.element.size?.width || 300, 
    height: props.element.size?.height || 3, 
    rotation: props.element.style?.rotation || (props.element.type === 'vline' ? 90 : 0) // 竖线默认90度，横线0度
});

watch(() => props.element, (newVal) => {
    form.color = newVal.style?.background || '#000';
    form.thickness = newVal.size?.height || 3;
    form.width = newVal.size?.width || 300;
    form.height = newVal.size?.height || 3;
    form.rotation = newVal.style?.rotation || (newVal.type === 'vline' ? 90 : 0);
}, { deep: true });
const updateStyle = () => {
    emits('update-element', {
        ...props.element,
        size: {
            ...props.element.size,
            height: form.thickness
        },
        style: {
            ...props.element.style,
            background: form.color,
            rotation: form.rotation 
        }
    });
};

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