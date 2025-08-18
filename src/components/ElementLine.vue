<template>
    <!-- <div class="element-line" :style="{
        width: `${element.size.width}px`,
        height: `${element.size.height}px`,
        backgroundColor: element.style?.background || '#000',
        transform: element.style?.transform ? element.style.transform : 'none'
    }"></div> -->
    <div class="element-line" :style="{
        width: `${element.size.width}px`,
        height: `${element.size.height}px`,
        backgroundColor: element.style?.background || '#000',
        transform: `rotate(${element.style?.rotation || 0}deg)`
    }"></div>
</template>

<script setup lang="ts">
defineProps({ element: Object });
</script>