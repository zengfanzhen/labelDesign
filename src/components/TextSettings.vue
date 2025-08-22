<template>
    <el-form class="text-settings" :model="form" label-width="80px" size="small">
        <!-- 1. 基本设置 -->
        <el-form-item label="内容">
            <el-input v-model="form.content" :disabled="isContentDisabled" type="textarea" :rows="3"
                placeholder="请输入文本内容" @change="updateElement" />
        </el-form-item>
        <el-form-item label="绑定变量">
            <el-select v-model="selectedVariable" placeholder="请选择变量" clearable @change="handleVariableChange">
                <el-option v-for="(label, key) in variables" :key="key" :label="label" :value="key" />
            </el-select>
        </el-form-item>
        <!-- 2. 字体样式 -->
        <el-form-item label="字体">
            <el-select v-model="form.fontFamily" @change="updateElement">
                <el-option label="微软雅黑" value="微软雅黑" />
                <el-option label="宋体" value="宋体" />
                <el-option label="Arial" value="Arial" />
            </el-select>
        </el-form-item>

        <el-form-item label="字体大小">
            <el-input v-model="form.fontSize" type="number" min="10" max="48" suffix="px" @change="updateElement" />
        </el-form-item>

        <el-form-item label="字体颜色">
            <el-color-picker v-model="form.color" size="small" @change="updateElement" />
        </el-form-item>



    </el-form>
</template>
<script setup lang="ts">
import { ref, reactive, watch, defineProps, defineEmits, computed } from 'vue';
// 接收父组件传递的当前文本元素
const props = defineProps({
    element: {
        type: Object,
        required: true,
        default: () => ({})
    }
});
//  触发父组件更新元素
const emits = defineEmits(['update-element']);
// 可用变量列表
const variables = reactive({
    inspectionTaskNumber: '检测任务编号',
    inspectionBatch: '检验批',
    materialName: '样品名称',
    productionBatch: '生产批号',
    samplingUser: '取样人',
    samplingTime: '取样时间',
    sampleStatus: '样品状态',
    storageCondition: '保存条件',
    samplePurpose: '样品用途'
});

//  表单状态（双向绑定设置项）
const form = reactive({
    content: '', // 文本内容（支持变量）
    inputType: 'text', // 输入类型：text/radio/checkbox/input
    fontFamily: '微软雅黑', // 字体
    fontSize: 14, // 字体大小（px）
    color: '#000000', // 字体颜色
    isBold: false, // 是否加粗
    isItalic: false, // 是否斜体
    isUnderline: false, // 是否下划线
    textAlign: 'left', // 对齐方式：left/center/right
    options: [] as Array<{ label: string; value: string | number }>,// 选项列表（单选/复选用）
    selectedVariableLabel: '' // 选中的变量label
});
// 用于跟踪当前选择的变量
const selectedVariable = ref<string>('');

// 计算属性：判断内容输入框是否应该被禁用
const isContentDisabled = computed(() => {
    return !!selectedVariable.value;
});
// 处理变量选择变化
const handleVariableChange = (variableKey: string | null) => {
    if (variableKey) {
        form.content = '${' + variableKey + '}';
        form.selectedVariableLabel = variables[variableKey] || '';
    } else {
        form.content = props.element.content || '';
        form.selectedVariableLabel = ''; // 清空label
    }
    updateElement();
};
// 检查当前内容是否匹配某个变量
const checkIfVariableSelected = () => {
    const content = props.element.content;
    if (content && content.startsWith('${') && content.endsWith('}')) {
        const variableKey = content.substring(2, content.length - 1);
        if (variables[variableKey]) {
            selectedVariable.value = variableKey;
            form.selectedVariableLabel = variables[variableKey] || '';
        }
    } else {
        selectedVariable.value = '';
        form.selectedVariableLabel = '';

    }
};
// 初始化表单：从当前元素中获取初始值
const initForm = () => {
    form.content = props.element.content || '';
    form.inputType = props.element.inputType || 'text';
    form.fontFamily = props.element.style?.fontFamily || '微软雅黑';
    form.fontSize = props.element.style?.fontSize ? parseInt(props.element.style.fontSize) : 14;
    form.color = props.element.style?.color || '#000000';
    form.isBold = props.element.style?.fontWeight === 'bold';
    form.isItalic = props.element.style?.fontStyle === 'italic';
    form.isUnderline = props.element.style?.textDecoration === 'underline';
    form.textAlign = props.element.style?.textAlign || 'left';
    form.options = props.element.options || [];

    // 检查当前内容是否匹配某个变量
    checkIfVariableSelected();
};
//  初始化表单（组件挂载时执行）
initForm();

watch(
    () => props.element,
    (newEl) => {
        if (newEl) {
            form.content = newEl.content || '';
            form.fontFamily = newEl.style?.fontFamily || '微软雅黑';
            form.fontSize = parseInt(newEl.style?.fontSize || '14');
            form.color = newEl.style?.color || '#000000';
            form.isBold = newEl.style?.fontWeight === 'bold';
            form.isItalic = newEl.style?.fontStyle === 'italic';
            form.isUnderline = newEl.style?.textDecoration === 'underline';
            form.textAlign = newEl.style?.textAlign || 'left';
            form.options = newEl.options || [];

            // 检查是否是变量
            checkIfVariableSelected();
        }
    },
    { deep: true, immediate: true }
);
//  更新元素：将表单状态同步到父组件
const updateElement = () => {
    const updatedStyle = {
        fontFamily: form.fontFamily,
        fontSize: form.fontSize,
        color: form.color,
        fontWeight: form.isBold ? 'bold' : 'normal',
        fontStyle: form.isItalic ? 'italic' : 'normal',
        textDecoration: form.isUnderline ? 'underline' : 'none',
        textAlign: form.textAlign
    };

    emits('update-element', {
        ...props.element,
        content: form.content,
        inputType: form.inputType,
        style: updatedStyle,
        // 👇 重点：当字体大小变化时，也更新元素高度，以更新缩放框大小
        // size: {
        //     ...props.element.size,
        //     height: form.fontSize + 20 // 适当加 padding
        // },
        options: form.options,
        selectedVariableLabel: form.selectedVariableLabel
    });
};

</script>
<style scoped>
.text-settings {
    padding: 10px 0;
}

/* 选项列表布局 */
.options-container {
    margin-top: 10px;
}

.option-item {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
    gap: 10px;
}

.option-input {
    flex: 1;
}

/* 变量列表布局 */
.variables-container {
    margin-top: 10px;
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
}
</style>