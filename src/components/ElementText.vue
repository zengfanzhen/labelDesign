<!-- ElementText.vue -->
<template>
    <div class="element-text" :style="dynamicStyle" @click.stop="handleClick" @dblclick.stop="startEditing"
        ref="textContainer">
        <span v-if="!isEditing">{{ displayContent }}</span>
        <input v-else type="text" v-model="editContent" @blur="stopEditing" @keydown.enter="stopEditing" autofocus
            ref="inputField" />
    </div>
</template>

<script setup lang="ts">
import { ref, defineProps, defineEmits, watch, onMounted, computed } from 'vue';
import { debounce } from 'lodash-es';

interface Size {
    width: number;
    height: number;
}

// 定义 props
const props = defineProps({
    element: {
        type: Object,
        required: true
    },
    size: {
        type: Object as () => Size,
        required: false,
        default: () => ({ width: 200, height: 30 })
    },
    variables: {
        type: Object,
        default: () => ({})
    }
});

onMounted(() => {
    if (props.element.content === '新文本') {
        startEditing();
    }
});

// 计算显示内容（处理变量替换）
const displayContent = computed(() => {
    let content = props.element.content || '';

    // 如果有传入的变量值，则替换内容中的变量
    if (props.variables && Object.keys(props.variables).length > 0) {
        Object.keys(props.variables).forEach(key => {
            const regex = new RegExp(`{{${key}}}`, 'g');
            content = content.replace(regex, props.variables[key]);
        });
    }

    return content;
});

const dynamicStyle = computed(() => {
    const fontSize = props.element.style?.fontSize || 14;
    return {
        fontSize: `${fontSize}px`,
        color: props.element.style?.color || '#000',
        fontWeight: props.element.style?.fontWeight || 'normal',
        fontStyle: props.element.style?.fontStyle || 'normal',
        textDecoration: props.element.style?.textDecoration || 'none'
    };
});

// 定义事件
const emits = defineEmits(['update-element']);

// 内部状态
const isEditing = ref(false);
const editContent = ref(props.element.content);

// 引用 DOM 元素（用于自动聚焦）
const inputField = ref<HTMLInputElement | null>(null);

// 开始编辑（双击触发）
const startEditing = () => isEditing.value = true;

// 停止编辑（失去焦点或回车）
const stopEditing = () => {
    isEditing.value = false;
    if (editContent.value !== props.element.content) {
        emits('update-element', {
            id: props.element.id,
            content: editContent.value,
            style: props.element.style
        });
    }
};

// 点击事件处理：单击进入编辑模式
const handleClick = () => {
    startEditing();
};

// 监听外部内容变化，保持同步
watch(
    () => props.element.content,
    (val) => {
        editContent.value = val;
    }
);

watch(
    () => editContent.value,
    debounce((newVal) => {
        if (newVal !== props.element.content) {
            emits('update-element', {
                id: props.element.id,
                content: newVal,
                style: props.element.style
            });
        }
    }, 300)
);

watch(
    () => props.element.style.fontSize,
    (newSize) => {
        const fontSizeNum = parseInt(newSize);
        if (!isNaN(fontSizeNum) && props.element.type === 'text') {
            emits('update-element', {
                id: props.element.id,
                size: {
                    ...props.element.size,
                    height: fontSizeNum + 20
                }
            });
        }
    }
);
</script>

<style scoped>
.element-text {
    height: 100%;
    display: flex;
    align-items: center;
    padding: 0 5px;
    position: relative;
    cursor: text;
    width: 100%;
}

.element-text span {
    pointer-events: none;
    /* 防止文字选中干扰拖拽 */
    width: 100%;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.element-text input {
    width: 100%;
    height: 100%;
    border: none;
    outline: none;
    padding: 0 5px;
    font-size: inherit;
    color: inherit;
    background: transparent;
}
</style>