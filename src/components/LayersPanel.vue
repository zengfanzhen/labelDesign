<template>
    <div class="layers-panel">
        <div v-for="(el, index) in elements" :key="el.id" class="layer-item"
            :class="{ selected: currentElement?.id === el.id }" @click="selectElement(el)">
            <el-checkbox v-model="el.visible" @change="updateElement(el)">
                {{ getLayerName(el) }}
            </el-checkbox>
            <div class="layer-actions">
                <el-button icon="ArrowUp" size="small" @click="moveUp(index)" :disabled="index === 0" />
                <el-button icon="ArrowDown" size="small" @click="moveDown(index)"
                    :disabled="index === elements.length - 1" />
                <el-button icon="Delete" size="small" type="danger" @click="deleteElement(el.id)" />
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { defineProps, defineEmits } from 'vue';
import { ArrowUp, ArrowDown, Delete } from '@element-plus/icons-vue';

const props = defineProps({ elements: Array, currentElement: Object });
const emits = defineEmits(['select-element', 'delete-element', 'reorder-elements']);

// 获取图层名称（简化显示）
// const getLayerName = (el: any) => `${el.type}: ${el.content.slice(0, 10)}...`;
const getLayerName = (el: any) => {
    switch (el.type) {
        case 'line': return '线条';
        case 'rectangle': return '矩形';
        default: return `${el.type}: ${el.content.slice(0, 10)}...`;
    }
};
// 选择元素
const selectElement = (el: any) => emits('select-element', el);

// 删除元素
const deleteElement = (id: string) => emits('delete-element', id);

// 上移图层
const moveUp = (index: number) => emits('reorder-elements', index, index - 1);

// 下移图层
const moveDown = (index: number) => emits('reorder-elements', index, index + 1);

// 更新元素（显示/隐藏）
const updateElement = (el: any) => emits('update-element', el);
</script>

<style scoped>
.layers-panel {
    padding: 10px;
}

.layer-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 5px 0;
    border-bottom: 1px solid #eee;
    cursor: pointer;
}

.layer-item.selected {
    background-color: #f0f5ff;
}

.layer-actions {
    display: flex;
    gap: 5px;
}
</style>