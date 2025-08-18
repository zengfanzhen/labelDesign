<template>
    <div class="table-element" :style="tableContainerStyle" @mouseenter="showResizers = true"
        @mouseleave="showResizers = false">
        <table :style="tableStyle">
            <tbody>
                <tr v-for="rowIndex in element.rows" :key="rowIndex" :style="getRowStyle(rowIndex)">
                    <td v-for="colIndex in element.cols" :key="colIndex" :style="getCellStyle(rowIndex, colIndex)"
                        :class="{ selected: isSelected(rowIndex, colIndex) }"
                        @mousedown="handleCellMouseDown(rowIndex, colIndex, $event)"
                        @mouseover="updateCellSelection(rowIndex, colIndex)" @mouseup="endCellSelection">
                    </td>
                </tr>
            </tbody>
        </table>
        <!-- 行高调整器 -->
        <div v-for="rowIndex in getInternalRowIndices()" :key="'row-resizer-' + rowIndex" v-show="showResizers"
            class="row-resizer" :style="getRowResizerStyle(rowIndex)"
            @mousedown.stop="startRowResize(rowIndex, $event)"></div>
        <!-- 列宽调整器 -->
        <div v-for="colIndex in getInternalColIndices()" :key="'col-resizer-' + colIndex" v-show="showResizers"
            class="col-resizer" :style="getColResizerStyle(colIndex)"
            @mousedown.stop="startColResize(colIndex, $event)"></div>

        <!-- 合并操作按钮 -->
        <div v-if="selectedCells.length > 1" class="merge-actions" @mousedown.stop>
            <button @click="mergeCells">合并单元格</button>
            <button @click="clearSelection">取消选择</button>
        </div>
        <!-- <div v-else-if="selectedCells.length > 1 && !canMerge" class="merge-actions" @mousedown.stop>
            <button @click="clearSelection">取消选择</button>
        </div> -->
    </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue';

interface TableElement {
    id: string;
    type: string;
    pos: { top: number; left: number };
    size: { width: number; height: number };
    rows: number;
    cols: number;
    width: number;
    height: number;
    rowHeights?: number[]; // 每行高度数组
    colWidths?: number[];  // 每列宽度数组
    borderColor?: string;
    borderWidth?: number;
    style?: Record<string, any>;
    mergedCells?: MergedCell[]; // 合并单元格信息
}

interface MergedCell {
    startRow: number;
    startCol: number;
    endRow: number;
    endCol: number;
}

interface Props {
    element: TableElement;
}

const props = defineProps<Props>();
const emits = defineEmits(['update-element']);

const showResizers = ref(false);

// 单元格选择相关状态
const isSelecting = ref(false);
const selectedCells = ref<{ row: number, col: number }[]>([]);
const selectionStart = ref<{ row: number, col: number } | null>(null);

// 行高调整状态
const rowResizeState = ref<{
    isResizing: boolean;
    rowIndex: number;
    startY: number;
    initialRowHeights: number[];
    initialHeight: number;
}>({
    isResizing: false,
    rowIndex: -1,
    startY: 0,
    initialRowHeights: [],
    initialHeight: 0
});

// 列宽调整状态
const colResizeState = ref<{
    isResizing: boolean;
    colIndex: number;
    startX: number;
    initialColWidths: number[];
    initialWidth: number;
}>({
    isResizing: false,
    colIndex: -1,
    startX: 0,
    initialColWidths: [],
    initialWidth: 0
});

// 计算是否可以合并选中的单元格
// const canMerge = computed(() => {
//     if (selectedCells.value.length < 2) return false;

//     // 检查选区是否为连续的矩形区域
//     const rows = selectedCells.value.map(cell => cell.row);
//     const cols = selectedCells.value.map(cell => cell.col);

//     const minRow = Math.min(...rows);
//     const maxRow = Math.max(...rows);
//     const minCol = Math.min(...cols);
//     const maxCol = Math.max(...cols);

//     // 检查是否所有应该被选中的单元格都被选中了（即形成一个完整的矩形）
//     const expectedCount = (maxRow - minRow + 1) * (maxCol - minCol + 1);
//     if (selectedCells.value.length !== expectedCount) return false;

//     // 检查选区是否跨越了现有的合并区域边界
//     if (props.element.mergedCells) {
//         for (const merged of props.element.mergedCells) {
//             // 检查选区是否与合并区域有部分重叠（跨越边界）
//             const mergeStartRow = merged.startRow;
//             const mergeEndRow = merged.endRow;
//             const mergeStartCol = merged.startCol;
//             const mergeEndCol = merged.endCol;

//             // 检查是否有部分重叠的情况
//             const overlapRow = (minRow <= mergeEndRow && maxRow >= mergeStartRow);
//             const overlapCol = (minCol <= mergeEndCol && maxCol >= mergeStartCol);

//             if (overlapRow && overlapCol) {
//                 // 如果有重叠，检查是否完全包含或完全不包含这个合并区域
//                 const completelyContains = (minRow <= mergeStartRow && maxRow >= mergeEndRow &&
//                     minCol <= mergeStartCol && maxCol >= mergeEndCol);

//                 const completelyExcludes = (maxRow < mergeStartRow || minRow > mergeEndRow ||
//                     maxCol < mergeStartCol || minCol > mergeEndCol);

//                 // 如果既不完全包含也不完全排除，则说明跨越了边界
//                 if (!completelyContains && !completelyExcludes) {
//                     return false;
//                 }
//             }
//         }
//     }

//     return true;
// });

// 监听行数/列数变化，重新分配行高/列宽
watch(() => [props.element.rows, props.element.cols], ([newRows, newCols], [oldRows, oldCols]) => {
    const updates: any = {};
    let needsUpdate = false;

    // 如果行数发生变化
    if (newRows !== oldRows && props.element.rowHeights) {
        const newRowHeights: number[] = [];
        const avgHeight = props.element.height / newRows;
        for (let i = 0; i < newRows; i++) {
            newRowHeights.push(avgHeight);
        }
        updates.rowHeights = newRowHeights;
        needsUpdate = true;
    }

    // 如果列数发生变化
    if (newCols !== oldCols && props.element.colWidths) {
        const newColWidths: number[] = [];
        const avgWidth = props.element.width / newCols;
        for (let i = 0; i < newCols; i++) {
            newColWidths.push(avgWidth);
        }
        updates.colWidths = newColWidths;
        needsUpdate = true;
    }

    if (needsUpdate) {
        emits('update-element', {
            id: props.element.id,
            ...updates
        });
    }
});

// 获取行索引
const getInternalRowIndices = () => {
    const indices = [];
    for (let i = 0; i < props.element.rows; i++) {
        indices.push(i);
    }
    return indices;
};

// 获取列索引
const getInternalColIndices = () => {
    const indices = [];
    for (let i = 0; i < props.element.cols; i++) {
        indices.push(i);
    }
    return indices;
};

const tableContainerStyle = computed(() => {
    return {
        width: props.element.width + 'px',
        height: props.element.height + 'px',
        position: 'absolute' as const,
        top: '0px',
        left: '0px'
    };
});

const tableStyle = computed(() => {
    const borderWidth = props.element.borderWidth !== undefined ? props.element.borderWidth : 1;
    const borderColor = props.element.borderColor || '#000000';

    return {
        width: props.element.width + 'px',
        height: props.element.height + 'px',
        border: `${borderWidth}px solid ${borderColor}`,
        borderCollapse: 'collapse' as const,
        tableLayout: 'fixed' as const
    };
});

// 获取行样式
const getRowStyle = (rowIndex: number) => {
    // 注意：rowIndex 从 1 开始
    const rowHeights = props.element.rowHeights ||
        Array(props.element.rows).fill(props.element.height / props.element.rows);

    return {
        height: (rowHeights[rowIndex - 1] || 0) + 'px'
    };
};

// 获取单元格样式
const getCellStyle = (rowIndex: number, colIndex: number) => {


    const borderWidth = props.element.borderWidth !== undefined ? props.element.borderWidth : 1;
    const borderColor = props.element.borderColor || '#000000';
    const colWidths = props.element.colWidths ||
        Array(props.element.cols).fill(props.element.width / props.element.cols);
    // 检查是否为合并单元格的一部分
    const mergedStyle: Record<string, any> = {};
    if (props.element.mergedCells) {
        console.log('mergedStyle', mergedStyle);
        console.log('props.element.mergedCells', props.element.mergedCells);
        for (const merged of props.element.mergedCells) {
            console.log('merged--------', merged);
            // 隐藏被合并的单元格（除了起始单元格）
            if (rowIndex > merged.startRow && rowIndex <= merged.endRow &&
                colIndex >= merged.startCol && colIndex <= merged.endCol) {
                console.log('mergedStyle', mergedStyle);
                mergedStyle.display = 'none';
            }
            // 起始单元格需要设置合并后的宽高
            else if (rowIndex === merged.startRow && colIndex === merged.startCol) {
                let width = 0;
                let height = 0;

                // 计算合并后的宽度
                // for (let i = merged.startCol - 1; i < merged.endCol; i++) {
                //     width += colWidths[i] || 0;
                // }
                for (let i = merged.startCol - 1; i < merged.endCol; i++) {
                    width += colWidths[i] || 0;
                }

                // 计算合并后的高度
                if (props.element.rowHeights) {
                    // for (let i = merged.startRow - 1; i < merged.endRow; i++) {
                    //     height += props.element.rowHeights[i] || 0;
                    // }
                    for (let i = 0; i < merged.endRow; i++) {
                        height += props.element.rowHeights[i] || 0;
                    }
                } else {
                    height = (props.element.height / props.element.rows) * (merged.endRow - merged.startRow + 1);
                }

                mergedStyle.width = width + 'px';
                mergedStyle.height = height + 'px';
            }
        }
    }

    return {
        border: `${borderWidth}px solid ${borderColor}`,
        padding: '0',
        margin: '0',
        width: (colWidths[colIndex - 1] || 0) + 'px',
        boxSizing: 'border-box' as const,
        ...mergedStyle
    };
};

// 检查单元格是否被选中
const isSelected = (rowIndex: number, colIndex: number) => {
    return selectedCells.value.some(cell =>
        cell.row === rowIndex && cell.col === colIndex
    );
};

// 处理单元格鼠标按下事件
const handleCellMouseDown = (rowIndex: number, colIndex: number, event: MouseEvent) => {
    // 如果按下了Ctrl键，则处理单元格选择
    if (event.ctrlKey || event.metaKey) {
        event.stopPropagation(); // 阻止事件冒泡，避免影响元素拖拽
        startCellSelection(rowIndex, colIndex);
    }
    // 否则让事件冒泡，允许父组件处理元素选择和拖拽
};

// 开始单元格选择
const startCellSelection = (rowIndex: number, colIndex: number) => {
    isSelecting.value = true;
    selectionStart.value = { row: rowIndex, col: colIndex };
    // 如果已经选择了单元格，则切换选择状态
    if (isSelected(rowIndex, colIndex)) {
        selectedCells.value = selectedCells.value.filter(
            cell => !(cell.row === rowIndex && cell.col === colIndex)
        );
    } else {
        selectedCells.value = [...selectedCells.value, { row: rowIndex, col: colIndex }];
    }
};

// 更新单元格选择（拖拽选择多个单元格）
const updateCellSelection = (rowIndex: number, colIndex: number) => {
    if (!isSelecting.value) return;

    if (selectionStart.value) {
        const startRow = selectionStart.value.row;
        const startCol = selectionStart.value.col;

        const minRow = Math.min(startRow, rowIndex);
        const maxRow = Math.max(startRow, rowIndex);
        const minCol = Math.min(startCol, colIndex);
        const maxCol = Math.max(startCol, colIndex);

        const newSelection = [];
        for (let r = minRow; r <= maxRow; r++) {
            for (let c = minCol; c <= maxCol; c++) {
                newSelection.push({ row: r, col: c });
            }
        }

        selectedCells.value = newSelection;
    }
};

// 结束单元格选择
const endCellSelection = () => {
    isSelecting.value = false;
};

// 清除单元格选择
const clearSelection = () => {
    selectedCells.value = [];
};

// 合并单元格
const mergeCells = () => {
    // if (!canMerge.value) return;
    // 计算选择区域的边界
    const rows = selectedCells.value.map(cell => cell.row);
    const cols = selectedCells.value.map(cell => cell.col);
    const startRow = Math.min(...rows);
    const endRow = Math.max(...rows);
    const startCol = Math.min(...cols);
    const endCol = Math.max(...cols);


    // 创建新的合并单元格信息
    const newMergedCell: MergedCell = {
        startRow,
        endRow,
        startCol,
        endCol
    };
    // 更新元素
    const mergedCells = props.element.mergedCells ? [...props.element.mergedCells] : [];
    mergedCells.push(newMergedCell);
    emits('update-element', {
        id: props.element.id,
        mergedCells
    });

    // 清空选择
    selectedCells.value = [];
};

// 获取行调整器样式
const getRowResizerStyle = (rowIndex: number) => {
    // rowIndex 从 0 开始，表示第 rowIndex 行和 rowIndex+1 行之间的分割线
    let top = 0;
    const rowHeights = props.element.rowHeights ||
        Array(props.element.rows).fill(props.element.height / props.element.rows);

    for (let i = 0; i <= rowIndex; i++) {
        top += rowHeights[i];
    }

    return {
        position: 'absolute',
        top: top + 'px',
        left: '0px',
        width: '100%',
        height: '5px',
        cursor: 'ns-resize',
        zIndex: 10
    };
};

// 获取列调整器样式
const getColResizerStyle = (colIndex: number) => {
    // colIndex 从 0 开始，表示第 colIndex 列和 colIndex+1 列之间的分割线
    let left = 0;
    const colWidths = props.element.colWidths ||
        Array(props.element.cols).fill(props.element.width / props.element.cols);
    for (let i = 0; i <= colIndex; i++) {
        left += colWidths[i];
    }
    return {
        position: 'absolute',
        top: '0px',
        left: left + 'px',
        width: '5px',
        height: '100%',
        cursor: 'ew-resize',
        zIndex: 10
    };
};

// 开始行高调整
const startRowResize = (rowIndex: number, e: MouseEvent) => {
    e.preventDefault();
    e.stopPropagation(); // 阻止事件冒泡
    // 获取当前每行的高度
    const currentRowHeights: number[] = [];
    if (props.element.rowHeights && props.element.rowHeights.length === props.element.rows) {
        currentRowHeights.push(...props.element.rowHeights);
    } else {
        const defaultHeight = props.element.height / props.element.rows;
        for (let i = 0; i < props.element.rows; i++) {
            currentRowHeights.push(defaultHeight);
        }
    }
    rowResizeState.value = {
        isResizing: true,
        rowIndex,
        startY: e.clientY,
        initialRowHeights: currentRowHeights,
        initialHeight: props.element.height
    };
    document.addEventListener('mousemove', onRowResize);
    document.addEventListener('mouseup', endRowResize);
};

// 行高调整中
const onRowResize = (e: MouseEvent) => {
    if (!rowResizeState.value.isResizing) return;

    const dy = e.clientY - rowResizeState.value.startY;
    const rowIndex = rowResizeState.value.rowIndex;

    // 计算新的行高
    const newRowHeights = [...rowResizeState.value.initialRowHeights];
    newRowHeights[rowIndex] = Math.max(20, rowResizeState.value.initialRowHeights[rowIndex] + dy);

    // 调整下一行高度以保持总高度不变
    if (rowIndex + 1 < newRowHeights.length) {
        const heightDiff = newRowHeights[rowIndex] - rowResizeState.value.initialRowHeights[rowIndex];
        newRowHeights[rowIndex + 1] = Math.max(20, rowResizeState.value.initialRowHeights[rowIndex + 1] - heightDiff);
    }

    // 计算新的总高度
    const newHeight = newRowHeights.reduce((sum, height) => sum + height, 0);

    emits('update-element', {
        id: props.element.id,
        rowHeights: newRowHeights,
        height: newHeight,
        size: {
            ...props.element.size,
            height: newHeight
        }
    });
};

// 结束行高调整
const endRowResize = () => {
    rowResizeState.value.isResizing = false;
    document.removeEventListener('mousemove', onRowResize);
    document.removeEventListener('mouseup', endRowResize);
};

// 开始列宽调整
const startColResize = (colIndex: number, e: MouseEvent) => {
    e.preventDefault();
    e.stopPropagation(); // 阻止事件冒泡

    // 获取当前每列的宽度
    const currentColWidths: number[] = [];
    if (props.element.colWidths && props.element.colWidths.length === props.element.cols) {
        currentColWidths.push(...props.element.colWidths);
    } else {
        const defaultWidth = props.element.width / props.element.cols;
        for (let i = 0; i < props.element.cols; i++) {
            currentColWidths.push(defaultWidth);
        }
    }

    colResizeState.value = {
        isResizing: true,
        colIndex,
        startX: e.clientX,
        initialColWidths: currentColWidths,
        initialWidth: props.element.width
    };

    document.addEventListener('mousemove', onColResize);
    document.addEventListener('mouseup', endColResize);
};

// 列宽调整中
const onColResize = (e: MouseEvent) => {
    if (!colResizeState.value.isResizing) return;

    const dx = e.clientX - colResizeState.value.startX;
    const colIndex = colResizeState.value.colIndex;

    // 计算新的列宽
    const newColWidths = [...colResizeState.value.initialColWidths];
    newColWidths[colIndex] = Math.max(20, colResizeState.value.initialColWidths[colIndex] + dx);

    // 调整下一列宽度以保持总宽度不变
    if (colIndex + 1 < newColWidths.length) {
        const widthDiff = newColWidths[colIndex] - colResizeState.value.initialColWidths[colIndex];
        newColWidths[colIndex + 1] = Math.max(20, colResizeState.value.initialColWidths[colIndex + 1] - widthDiff);
    }

    // 计算新的总宽度
    const newWidth = newColWidths.reduce((sum, width) => sum + width, 0);

    emits('update-element', {
        id: props.element.id,
        colWidths: newColWidths,
        width: newWidth,
        size: {
            ...props.element.size,
            width: newWidth
        }
    });
};

// 结束列宽调整
const endColResize = () => {
    colResizeState.value.isResizing = false;
    document.removeEventListener('mousemove', onColResize);
    document.removeEventListener('mouseup', endColResize);
};
</script>

<style scoped>
.table-element {
    pointer-events: auto;
    width: 100%;
    height: 100%;
}

table {
    border-spacing: 0;
    width: 100%;
    height: 100%;
    box-sizing: border-box;
}

td {
    padding: 0;
    margin: 0;
    overflow: hidden;
    box-sizing: border-box;
    position: relative;
}

td.selected {
    background-color: rgba(64, 158, 255, 0.2);
}

.row-resizer,
.col-resizer {
    position: absolute;
    background-color: transparent;
    pointer-events: auto;
}

.row-resizer:hover,
.col-resizer:hover {
    background-color: rgba(30, 144, 255, 0.5);
}

.merge-actions {
    position: absolute;
    top: -40px;
    right: 0;
    display: flex;
    gap: 5px;
    align-items: center;
}

.merge-actions button {
    padding: 5px 10px;
    background-color: #409eff;
    color: white;
    border: none;
    border-radius: 3px;
    cursor: pointer;
}

.merge-actions button:hover {
    background-color: #66b1ff;
}

.merge-error {
    color: #f56c6c;
    font-size: 12px;
}
</style>