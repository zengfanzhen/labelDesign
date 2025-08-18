<!-- src/components/TableSettings.vue -->
<template>
    <div class="table-settings">
        <el-form :model="tableSettings" label-width="80px">
            <el-form-item label="行数">
                <el-input-number v-model="tableSettings.rows" :min="1" :max="50" @change="handleRowsChange" />
            </el-form-item>

            <el-form-item label="列数">
                <el-input-number v-model="tableSettings.cols" :min="1" :max="50" @change="handleColsChange" />
            </el-form-item>

            <el-form-item label="表格宽度">
                <el-input-number v-model="tableSettings.width" :min="20" :max="2000" @change="updateTableSettings" /> px
            </el-form-item>

            <el-form-item label="表格高度">
                <el-input-number v-model="tableSettings.height" :min="20" :max="2000" @change="updateTableSettings" />
                px
            </el-form-item>

            <!-- <el-form-item label="行高">
                <el-input-number v-model="tableSettings.rowHeight" :min="10" :max="500" @change="updateTableSettings" />
                px
            </el-form-item>

            <el-form-item label="列宽">
                <el-input-number v-model="tableSettings.colWidth" :min="10" :max="500" @change="updateTableSettings" />
                px
            </el-form-item> -->

            <el-form-item label="边框颜色">
                <el-color-picker v-model="tableSettings.borderColor" @change="updateTableSettings" />
            </el-form-item>

            <el-form-item label="边框宽度">
                <el-input-number v-model="tableSettings.borderWidth" :min="0" :max="10" @change="updateTableSettings" />
                px
            </el-form-item>
        </el-form>
    </div>
</template>

<script setup lang="ts">
import { ref, watch, reactive } from 'vue';
import { defineProps, defineEmits } from 'vue';

interface TableElement {
    id: string;
    type: string;
    pos: { top: number; left: number };
    size: { width: number; height: number };
    rows: number;
    cols: number;
    width: number;
    height: number;
    rowHeight: number;
    colWidth: number;
    rowHeights?: number[];
    colWidths?: number[];
    borderColor?: string;
    borderWidth?: number;
    style?: Record<string, any>;
}

interface Props {
    element: TableElement;
}

const props = defineProps<Props>();
const emits = defineEmits(['update-element']);

// 初始化表格设置
const tableSettings = reactive({
    rows: props.element.rows || 3,
    cols: props.element.cols || 3,
    width: props.element.width || 300,
    height: props.element.height || 150,
    rowHeight: props.element.rowHeight || 50,
    colWidth: props.element.colWidth || 100,
    borderColor: props.element.borderColor || '#000000',
    borderWidth: props.element.borderWidth || 1
});

// 监听属性变化
watch(() => props.element, (newElement) => {
    if (newElement) {
        tableSettings.rows = newElement.rows || 3;
        tableSettings.cols = newElement.cols || 3;
        tableSettings.width = newElement.width || 300;
        tableSettings.height = newElement.height || 150;
        tableSettings.rowHeight = newElement.rowHeight || 50;
        tableSettings.colWidth = newElement.colWidth || 100;
        tableSettings.borderColor = newElement.borderColor || '#000000';
        tableSettings.borderWidth = newElement.borderWidth || 1;
    }
}, { deep: true });

// 处理行数变化
const handleRowsChange = (newRows: number | undefined) => {
    if (newRows === undefined) return;

    const updatedElement: any = {
        id: props.element.id,
        rows: newRows,
        cols: tableSettings.cols,
        width: tableSettings.width,
        height: tableSettings.height,
        rowHeight: tableSettings.height / newRows,
        colWidth: tableSettings.colWidth,
        size: {
            width: tableSettings.width,
            height: tableSettings.height
        },
        borderColor: tableSettings.borderColor,
        borderWidth: tableSettings.borderWidth
    };

    // 如果增加行数，重新分配行高
    if (newRows > (props.element.rows || 3)) {
        // 生成新的行高数组，平均分配高度
        const newRowHeights: number[] = [];
        const avgHeight = tableSettings.height / newRows;
        for (let i = 0; i < newRows; i++) {
            newRowHeights.push(avgHeight);
        }
        updatedElement.rowHeights = newRowHeights;
    }
    // 如果减少行数，也需要重新分配行高
    else if (props.element.rowHeights) {
        const newRowHeights: number[] = [];
        const avgHeight = tableSettings.height / newRows;
        for (let i = 0; i < newRows; i++) {
            newRowHeights.push(avgHeight);
        }
        updatedElement.rowHeights = newRowHeights;
    }

    emits('update-element', updatedElement);
};

// 处理列数变化
const handleColsChange = (newCols: number | undefined) => {
    if (newCols === undefined) return;

    const updatedElement: any = {
        id: props.element.id,
        rows: tableSettings.rows,
        cols: newCols,
        width: tableSettings.width,
        height: tableSettings.height,
        rowHeight: tableSettings.rowHeight,
        colWidth: tableSettings.width / newCols,
        size: {
            width: tableSettings.width,
            height: tableSettings.height
        },
        borderColor: tableSettings.borderColor,
        borderWidth: tableSettings.borderWidth
    };

    // 如果增加列数，重新分配列宽
    if (newCols > (props.element.cols || 3)) {
        // 生成新的列宽数组，平均分配宽度
        const newColWidths: number[] = [];
        const avgWidth = tableSettings.width / newCols;
        for (let i = 0; i < newCols; i++) {
            newColWidths.push(avgWidth);
        }
        updatedElement.colWidths = newColWidths;
    }
    // 如果减少列数，也需要重新分配列宽
    else if (props.element.colWidths) {
        const newColWidths: number[] = [];
        const avgWidth = tableSettings.width / newCols;
        for (let i = 0; i < newCols; i++) {
            newColWidths.push(avgWidth);
        }
        updatedElement.colWidths = newColWidths;
    }

    emits('update-element', updatedElement);
};

// 更新表格设置（除行数和列数外的其他设置）
const updateTableSettings = () => {
    const updatedElement = {
        id: props.element.id,
        rows: tableSettings.rows,
        cols: tableSettings.cols,
        width: tableSettings.width,
        height: tableSettings.height,
        rowHeight: tableSettings.height / tableSettings.rows,
        colWidth: tableSettings.width / tableSettings.cols,
        size: {
            width: tableSettings.width,
            height: tableSettings.height
        },
        borderColor: tableSettings.borderColor,
        borderWidth: tableSettings.borderWidth
    };

    // 重新计算行高数组
    if (props.element.rowHeights) {
        const newRowHeights: number[] = [];
        const avgHeight = tableSettings.height / tableSettings.rows;
        for (let i = 0; i < tableSettings.rows; i++) {
            newRowHeights.push(avgHeight);
        }
        updatedElement.rowHeights = newRowHeights;
    }

    // 重新计算列宽数组
    if (props.element.colWidths) {
        const newColWidths: number[] = [];
        const avgWidth = tableSettings.width / tableSettings.cols;
        for (let i = 0; i < tableSettings.cols; i++) {
            newColWidths.push(avgWidth);
        }
        updatedElement.colWidths = newColWidths;
    }

    emits('update-element', updatedElement);
};
</script>

<style scoped>
.table-settings {
    padding: 10px 0;
}

.el-form-item {
    margin-bottom: 15px;
}
</style>