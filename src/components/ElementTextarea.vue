<template>
    <div class="element-textarea" :style="elementStyle">
        <textarea v-if="isEditing" ref="textareaRef" v-model="localContent" :style="textareaStyle" @blur="finishEditing"
            @keydown="handleKeydown" />
        <div v-else class="textarea-display" @dblclick="startEditing" :style="displayStyle">
            {{ displayContent }}
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick, onMounted, onBeforeUnmount } from 'vue';

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

const isEditing = ref(false);
const localContent = ref(props.element.content || '');
const textareaRef = ref<HTMLTextAreaElement | null>(null);

const elementStyle = computed(() => ({
    width: `${props.element.size.width}px`,
    height: `${props.element.size.height}px`,
    position: 'relative',
    overflow: 'hidden'
}));

const baseStyle = computed(() => ({
    fontFamily: props.element.style?.fontFamily || 'Arial',
    fontSize: `${props.element.style?.fontSize || 14}px`,
    fontWeight: props.element.style?.fontWeight || 'normal',
    fontStyle: props.element.style?.fontStyle || 'normal',
    textDecoration: props.element.style?.textDecoration || 'none',
    color: props.element.style?.color || '#000',
    backgroundColor: props.element.style?.backgroundColor || 'transparent',
    textAlign: props.element.style?.textAlign || 'left'
}));

const textareaStyle = computed(() => ({
    ...baseStyle.value,
    width: '100%',
    height: '100%',
    border: 'none',
    outline: 'none',
    resize: 'none',
    boxSizing: 'border-box',
    padding: '5px'
}));

const displayStyle = computed(() => ({
    ...baseStyle.value,
    width: '100%',
    height: '100%',
    padding: '5px',
    whiteSpace: 'pre-wrap',
    wordWrap: 'break-word',
    overflow: 'hidden',
    boxSizing: 'border-box'
}));

const displayContent = computed(() => {
    return props.element.content || '双击编辑多行文本';
});

const startEditing = () => {
    isEditing.value = true;
    localContent.value = props.element.content || '';
    nextTick(() => {
        textareaRef.value?.focus();
    });
};

const finishEditing = () => {
    isEditing.value = false;
    emits('update-element', {
        id: props.element.id,
        content: localContent.value
    });
};

const handleKeydown = (e: KeyboardEvent) => {
    if (e.key === 'Escape') {
        localContent.value = props.element.content || '';
        isEditing.value = false;
    }
};

const handleClickOutside = (e: MouseEvent) => {
    if (isEditing.value && textareaRef.value && !textareaRef.value.contains(e.target as Node)) {
        finishEditing();
    }
};

onMounted(() => {
    // 点击组件外部时结束编辑
    document.addEventListener('click', handleClickOutside);
});

onBeforeUnmount(() => {
    document.removeEventListener('click', handleClickOutside);
});
</script>

<style scoped>
.element-textarea {
    cursor: text;
    width: 100%;
    height: 100%;
}
</style>