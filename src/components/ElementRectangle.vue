<!-- src/components/ElementRectangle.vue -->
<template>
    <div class="element-rectangle" :style="rectangleStyle">
    </div>
</template>

<script setup lang="ts">
import { computed, defineProps } from 'vue';

interface ElementBase {
    id: string;
    type: string;
    pos: { top: number; left: number };
    size: { width: number; height: number };
    style?: Record<string, any>;
}

interface Props {
    element: ElementBase;
}

const props = defineProps<Props>();

const rectangleStyle = computed(() => {
    const style: Record<string, string> = {
        width: '100%',
        height: '100%',
        border: props.element.style?.border || '1px solid #000',
        backgroundColor: props.element.style?.background || 'transparent',
    };

    // 如果有旋转属性，添加旋转变换
    const rotation = props.element.style?.rotation || 0;
    if (rotation !== 0) {
        style.transform = `rotate(${rotation}deg)`;
        style.transformOrigin = 'center center';
    }

    return style;
});
</script>

<style scoped>
.element-rectangle {
    box-sizing: border-box;
}
</style>