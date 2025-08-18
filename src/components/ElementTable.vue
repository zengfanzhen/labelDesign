<template>
    <div class="table-element" :style="tableContainerStyle" @mouseenter="showResizers = true"
        @mouseleave="showResizers = false">
        <table :style="tableStyle">
            <tbody>
                <tr v-for="rowIndex in element.rows" :key="rowIndex" :style="getRowStyle(rowIndex)">
                    <td v-for="colIndex in element.cols" :key="colIndex" :style="getCellStyle(rowIndex, colIndex)"
                        :class="{ selected: isSelected(rowIndex, colIndex) }" :rowspan="getRowspan(rowIndex, colIndex)"
                        :colspan="getColspan(rowIndex, colIndex)"
                        @mousedown="handleCellMouseDown(rowIndex, colIndex, $event)">
                    </td>
                </tr>
            </tbody>
        </table>
        <!-- 行高调整器 -->
        <div v-for="rowIndex in getInternalRowIndices()" :key="'row-resizer-' + rowIndex" v-show="showResizers"
            class="row-resizer" :style="getRowResizerStyle(rowIndex)"
            @mousedown.stop="startRowResize(rowIndex, $event)">
        </div>
        <!-- 列宽调整器 -->
        <div v-for="colIndex in getInternalColIndices()" :key="'col-resizer-' + colIndex" v-show="showResizers"
            class="col-resizer" :style="getColResizerStyle(colIndex)"
            @mousedown.stop="startColResize(colIndex, $event)">
        </div>

        <!-- 合并操作按钮 -->
        <div v-if="canMerge" class="merge-actions" @mousedown.stop>
            <button @click="mergeCells">合并单元格</button>
            <button @click="clearSelection">取消选择</button>
        </div>
        <div v-else-if="selectedCells.length > 1 && !canMerge" class="merge-actions" @mousedown.stop>
            <!-- <span class="merge-error">无法合并选中单元格</span> -->
            <button @click="clearSelection">取消选择</button>
        </div>
        <!-- 拆分操作按钮 -->
        <div v-else-if="canSplit" class="merge-actions" @mousedown.stop>
            <button @click="splitCell">拆分单元格</button>
            <button @click="clearSelection">取消选择</button>
        </div>
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
    colspan?: number;
    rowspan?: number;
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
const canMerge = computed(() => {
    if (selectedCells.value.length < 2) return false;
    // 检查选区是否为连续的矩形区域
    const rows = selectedCells.value.map(cell => cell.row);
    const cols = selectedCells.value.map(cell => cell.col);
    const minRow = Math.min(...rows);
    const maxRow = Math.max(...rows);
    const minCol = Math.min(...cols);
    const maxCol = Math.max(...cols);
    // 检查是否所有应该被选中的单元格都被选中了（即形成一个完整的矩形）
    const expectedCount = (maxRow - minRow + 1) * (maxCol - minCol + 1);
    // console.log('expectedCount:', expectedCount);
    if (selectedCells.value.length !== expectedCount) return false;
    return true;
});
const canSplit = computed(() => {
    // 只有选中一个单元格时才可能进行拆分操作
    if (selectedCells.value.length !== 1) return false;

    const { row, col } = selectedCells.value[0];

    // 检查该单元格是否为合并单元格的起始单元格
    if (props.element.mergedCells) {
        return props.element.mergedCells.some(merged =>
            merged.startRow === row && merged.startCol === col &&
            (merged.rowspan > 1 || merged.colspan > 1)
        );
    }

    return false;
});


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
    const rowHeights = props.element.rowHeights ||
        Array(props.element.rows).fill(props.element.height / props.element.rows);

    return {
        height: (rowHeights[rowIndex - 1] || 0) + 'px'
    };
};

// 获取单元格的rowspan属性（是否跨行）
const getRowspan = (rowIndex: number, colIndex: number) => {
    // console.log('getRowspan:', rowIndex, colIndex);
    if (props.element.mergedCells) {
        for (const merged of props.element.mergedCells) {
            if (rowIndex === merged.startRow && colIndex === merged.startCol) {
                console.log('row--merged:', merged);
                return merged.endRow - merged.startRow + 1;
            }
        }
    }
    return 1;
};

// 获取单元格的colspan属性（是否跨列）
const getColspan = (rowIndex: number, colIndex: number) => {
    // console.log('getColspan:', rowIndex, colIndex);
    if (props.element.mergedCells) {
        for (const merged of props.element.mergedCells) {
            if (rowIndex === merged.startRow && colIndex === merged.startCol) {
                // console.log('col--merged:', merged);
                return merged.endCol - merged.startCol + 1;
            }
        }
    }
    return 1;
};

// 获取单元格样式
const getCellStyle = (rowIndex: number, colIndex: number) => {
    // console.log('getCellStyle:', rowIndex, colIndex);
    //基础样式参数获取
    const borderWidth = props.element.borderWidth !== undefined ? props.element.borderWidth : 1;
    const borderColor = props.element.borderColor || '#000000';
    const colWidths = props.element.colWidths ||
        Array(props.element.cols).fill(props.element.width / props.element.cols);
    // console.log('colWidths:', colWidths);
    // 检查是否为被合并的单元格（需要隐藏的）
    if (props.element.mergedCells) {
        for (const merged of props.element.mergedCells) {
            // 如果不是起始单元格，但在这个合并区域内，则隐藏
            if (!(rowIndex === merged.startRow && colIndex === merged.startCol) &&
                rowIndex >= merged.startRow && rowIndex <= merged.endRow &&
                colIndex >= merged.startCol && colIndex <= merged.endCol) {
                return {
                    display: 'none'
                };
            }
        }
    }
    // 计算单元格宽度（考虑合并情况）
    let cellWidth = colWidths[colIndex - 1] || 0;
    if (props.element.mergedCells) {
        for (const merged of props.element.mergedCells) {
            if (rowIndex === merged.startRow && colIndex === merged.startCol) {
                // 对于合并的起始单元格，计算总宽度
                cellWidth = 0;
                for (let i = merged.startCol; i <= merged.endCol; i++) {
                    cellWidth += colWidths[i - 1] || 0;
                }
                break;
            }
        }
    }
    return {
        border: `${borderWidth}px solid ${borderColor}`,
        padding: '0',
        margin: '0',
        width: cellWidth + 'px',
        boxSizing: 'border-box' as const
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
};

// 开始单元格选择
const startCellSelection = (rowIndex: number, colIndex: number) => {
    // 如果点击的是合并单元格的一部分，则选中整个合并区域
    let targetRow = rowIndex;
    let targetCol = colIndex;

    if (props.element.mergedCells) {
        for (const merged of props.element.mergedCells) {
            if (rowIndex >= merged.startRow && rowIndex <= merged.endRow &&
                colIndex >= merged.startCol && colIndex <= merged.endCol) {
                targetRow = merged.startRow;
                targetCol = merged.startCol;
                break;
            }
        }
    }

    isSelecting.value = true;
    selectionStart.value = { row: targetRow, col: targetCol };

    // 如果已经选择了单元格，则切换选择状态
    if (isSelected(targetRow, targetCol)) {
        selectedCells.value = selectedCells.value.filter(
            cell => !(cell.row === targetRow && cell.col === targetCol)
        );
    } else {
        selectedCells.value = [...selectedCells.value, { row: targetRow, col: targetCol }];
    }
};

// 更新单元格选择（拖拽选择多个单元格）
const updateCellSelection = (rowIndex: number, colIndex: number) => {
    if (!isSelecting.value || !selectionStart.value) return;

    const startRow = selectionStart.value.row;
    const startCol = selectionStart.value.col;

    const minRow = Math.min(startRow, rowIndex);
    const maxRow = Math.max(startRow, rowIndex);
    const minCol = Math.min(startCol, colIndex);
    const maxCol = Math.max(startCol, colIndex);

    const newSelection = [];
    for (let r = minRow; r <= maxRow; r++) {
        for (let c = minCol; c <= maxCol; c++) {
            // 如果单元格是合并单元格的一部分，则添加整个合并区域
            let cellToAdd = { row: r, col: c };

            if (props.element.mergedCells) {
                for (const merged of props.element.mergedCells) {
                    if (r >= merged.startRow && r <= merged.endRow &&
                        c >= merged.startCol && c <= merged.endCol) {
                        cellToAdd = { row: merged.startRow, col: merged.startCol };
                        break;
                    }
                }
            }

            // 避免重复添加
            if (!newSelection.some(cell => cell.row === cellToAdd.row && cell.col === cellToAdd.col)) {
                newSelection.push(cellToAdd);
            }
        }
    }

    selectedCells.value = newSelection;
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
    if (!canMerge.value) return;

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
        endCol,
        colspan: endCol - startCol + 1,
        rowspan: endRow - startRow + 1
    };
    console.log('newMergedCell', newMergedCell);
    // 更新元素
    const mergedCells = props.element.mergedCells ? [...props.element.mergedCells] : [];
    console.log('mergedCells11111', mergedCells);
    mergedCells.push(newMergedCell);
    console.log('mergedCells22222', mergedCells);
    emits('update-element', {
        id: props.element.id,
        mergedCells
    });

    // 清空选择
    selectedCells.value = [];
};
// 拆分单元格
const splitCell = () => {
    if (!canSplit.value) return;

    const { row, col } = selectedCells.value[0];
    let mergedCellIndex = -1;

    // 找到要拆分的合并单元格
    if (props.element.mergedCells) {
        mergedCellIndex = props.element.mergedCells.findIndex(merged =>
            merged.startRow === row && merged.startCol === col
        );
    }

    if (mergedCellIndex === -1) return;

    // 创建新的合并单元格数组，移除要拆分的单元格
    const newMergedCells = [...(props.element.mergedCells || [])];
    newMergedCells.splice(mergedCellIndex, 1);

    // 更新元素
    emits('update-element', {
        id: props.element.id,
        mergedCells: newMergedCells
    });

    // 清空选择
    clearSelection();
};
// 获取行调整器样式
const getRowResizerStyle = (rowIndex: number) => {
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
    e.stopPropagation();

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

    const newRowHeights = [...rowResizeState.value.initialRowHeights];
    newRowHeights[rowIndex] = Math.max(20, rowResizeState.value.initialRowHeights[rowIndex] + dy);

    if (rowIndex + 1 < newRowHeights.length) {
        const heightDiff = newRowHeights[rowIndex] - rowResizeState.value.initialRowHeights[rowIndex];
        newRowHeights[rowIndex + 1] = Math.max(20, rowResizeState.value.initialRowHeights[rowIndex + 1] - heightDiff);
    }

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
    e.stopPropagation();

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

    const newColWidths = [...colResizeState.value.initialColWidths];
    newColWidths[colIndex] = Math.max(20, colResizeState.value.initialColWidths[colIndex] + dx);

    if (colIndex + 1 < newColWidths.length) {
        const widthDiff = newColWidths[colIndex] - colResizeState.value.initialColWidths[colIndex];
        newColWidths[colIndex + 1] = Math.max(20, colResizeState.value.initialColWidths[colIndex + 1] - widthDiff);
    }

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