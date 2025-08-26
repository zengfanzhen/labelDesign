<template>
    <el-form :model="localElement" label-width="80px" @submit.prevent>
        <el-form-item label="内容">
            <el-input type="textarea" v-model="localElement.content" :rows="4" @change="updateContent"
                @input="updateContent" placeholder="请输入多行文本内容" />
        </el-form-item>

        <el-form-item label="字体">
            <el-select v-model="localElement.style.fontFamily" @change="updateStyle" placeholder="选择字体">
                <el-option label="Arial" value="Arial" />
                <el-option label="微软雅黑" value="Microsoft YaHei" />
                <el-option label="宋体" value="SimSun" />
                <el-option label="黑体" value="SimHei" />
                <el-option label="Helvetica" value="Helvetica" />
                <el-option label="Times New Roman" value="Times New Roman" />
            </el-select>
        </el-form-item>

        <el-form-item label="字体大小">
            <el-slider v-model="localElement.style.fontSize" :min="8" :max="72" show-input @change="updateStyle" />
        </el-form-item>

        <el-form-item label="字体样式">
            <el-checkbox v-model="isBold" @change="updateFontWeight">
                粗体
            </el-checkbox>
            <el-checkbox v-model="isItalic" @change="updateFontStyle">
                斜体
            </el-checkbox>
        </el-form-item>

        <el-form-item label="文本颜色">
            <el-color-picker v-model="localElement.style.color" @change="updateStyle" show-alpha />
        </el-form-item>

        <el-form-item label="背景色">
            <el-color-picker v-model="localElement.style.backgroundColor" @change="updateStyle" show-alpha />
        </el-form-item>

        <el-form-item label="对齐方式">
            <el-radio-group v-model="localElement.style.textAlign" @change="updateStyle">
                <el-radio-button label="left">左对齐</el-radio-button>
                <el-radio-button label="center">居中</el-radio-button>
                <el-radio-button label="right">右对齐</el-radio-button>
            </el-radio-group>
        </el-form-item>
    </el-form>
</template>

<script setup lang="ts">
import { ref, reactive, watch, computed } from 'vue';

interface Element {
    id: string;
    type: string;
    content?: string;
    style?: Record<string, any>;
    size: { width: number; height: number };
}

interface Props {
    element: Element;
}

const props = defineProps<Props>();
const emits = defineEmits(['update-element']);

// 创建本地副本以避免直接修改props
const localElement = reactive({
    id: props.element.id,
    type: props.element.type,
    content: props.element.content || '',
    style: {
        fontFamily: props.element.style?.fontFamily || 'Arial',
        fontSize: props.element.style?.fontSize || 14,
        fontWeight: props.element.style?.fontWeight || 'normal',
        fontStyle: props.element.style?.fontStyle || 'normal',
        textDecoration: props.element.style?.textDecoration || 'none',
        color: props.element.style?.color || '#000000',
        backgroundColor: props.element.style?.backgroundColor || 'transparent',
        textAlign: props.element.style?.textAlign || 'left'
    },
    size: {
        width: props.element.size.width,
        height: props.element.size.height
    }
});

// 计算属性用于字体样式复选框
const isBold = computed({
    get: () => localElement.style.fontWeight === 'bold',
    set: (val) => {
        localElement.style.fontWeight = val ? 'bold' : 'normal';
        updateStyle();
    }
});

const isItalic = computed({
    get: () => localElement.style.fontStyle === 'italic',
    set: (val) => {
        localElement.style.fontStyle = val ? 'italic' : 'normal';
        updateStyle();
    }
});

// 监听props变化并更新本地副本
watch(() => props.element, (newVal) => {
    if (newVal) {
        localElement.id = newVal.id;
        localElement.type = newVal.type;
        localElement.content = newVal.content || '';
        localElement.style = {
            fontFamily: newVal.style?.fontFamily || 'Arial',
            fontSize: newVal.style?.fontSize || 14,
            fontWeight: newVal.style?.fontWeight || 'normal',
            fontStyle: newVal.style?.fontStyle || 'normal',
            textDecoration: newVal.style?.textDecoration || 'none',
            color: newVal.style?.color || '#000000',
            backgroundColor: newVal.style?.backgroundColor || 'transparent',
            textAlign: newVal.style?.textAlign || 'left'
        };
        localElement.size = {
            width: newVal.size.width,
            height: newVal.size.height
        };
    }
}, { deep: true });

// 更新内容
const updateContent = () => {
    emits('update-element', {
        id: localElement.id,
        content: localElement.content
    });
};

// 更新样式
const updateStyle = () => {
    emits('update-element', {
        id: localElement.id,
        style: { ...localElement.style }
    });
};

// 更新字体粗细
const updateFontWeight = () => {
    emits('update-element', {
        id: localElement.id,
        style: { ...localElement.style }
    });
};

// 更新字体样式
const updateFontStyle = () => {
    emits('update-element', {
        id: localElement.id,
        style: { ...localElement.style }
    });
};
</script>

<style scoped>
.el-form-item {
    margin-bottom: 15px;
}

.el-slider {
    width: 100%;
}

.el-textarea {
    min-height: 80px;
}
</style>