<template>
    <div class="element-rectangle" :style="{
        width: `${element.size.width}px`,
        height: `${element.size.height}px`,
        border: element.style?.border || '1px solid #000',
        backgroundColor: element.style?.background || 'transparent'
    }"></div>
</template>

<script setup lang="ts">
defineProps({ element: Object });
</script>