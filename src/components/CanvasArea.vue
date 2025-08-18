<template>
    <div class="canvas-area">
        <div class="label-container" :style="{
            width: `${labelSettings.width}px`,
            height: `${labelSettings.height}px`,
            overflow: labelSettings.overflow ? 'hidden' : 'visible',
            backgroundSize: 'cover'
            }" @click="handleCanvasClick">
            <!-- 元素列表（循环渲染） -->
            <div v-for="(el, index) in elements" :key="el.id" class="element"
                :class="{ selected: props.currentElement?.id === el.id }" :style="{
                    position: 'absolute',
                    top: `${el.pos.top}px`,
                    left: `${el.pos.left}px`,
                    width: `${el.size.width}px`,
                    height: `${el.size.height}px`,
                    zIndex: index,
                    ...(props.currentElement?.id === el.id &&
                        el.type !== 'line' &&
                        el.type !== 'vline' && el.type !== 'table' ? selectedElementStyle(el) : {})
                }" @mousedown="startDrag(el, $event)" @click.stop="selectElement(el)">
                <!-- 根据元素类型渲染对应组件 -->
                <!-- <component :is="getComponent(el.type)" :element="el"
                    @update-element="$emit('update-element', $event)" /> -->
                <component :is="getComponent(el.type)" :element="el" :variables="dataVariables"
                    @update-element="$emit('update-element', $event)" />
                <!-- 缩放手柄 -->
                <div v-if="currentElementId === el.id && el.type !== 'table'" class="selection-container"
                    :style="getSelectionStyle(el)">
                    <div v-for="direction in ['nw', 'n', 'ne', 'e', 'se', 's', 'sw', 'w']" :key="direction"
                        class="handle" :class="`handle-${direction}`" :style="handleStyle(el, direction)"
                        @mousedown.stop="startResize(el, direction, $event)"></div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, nextTick, watch, computed, onUnmounted, onMounted } from 'vue';
import ElementText from './ElementText.vue';
import ElementBarcode from './ElementBarcode.vue';
import ElementQrcode from './ElementQrcode.vue';
import ElementLine from './ElementLine.vue';
import ElementVLine from './ElementVLine.vue';
import ElementRectangle from './ElementRectangle.vue';
import ElementTable from './ElementTable.vue';

// ========== 数据类型定义 ==========
interface Position {
    top: number;
    left: number;
}

interface Size {
    width: number;
    height: number;
}

interface ElementBase {
    id: string;
    type: string;
    pos: Position;
    size: Size;
    content?: string;
    style?: Record<string, any>;
    visible?: boolean;
}

interface Props {
    currentTool: string | null;
    labelSettings: Record<string, any>;
    elements: ElementBase[];
    currentElement: ElementBase | null; 
    dataVariables?: Record<string, string>; 
}
const props = defineProps<Props>();
const resizeDirection = ref<string>('');

const currentElementId = computed(() => props.currentElement?.id);

const emits = defineEmits<{
    (e: 'select-element', element: ElementBase | null): void;
    (e: 'update-element', payload: { id: string; pos?: Position; size?: Size; content?: string, style?: any }): void;
    (e: 'add-element', element: ElementBase): void;
}>();
// 在 setup 中引入 isAddingMode 状态
const isAddingMode = ref(false);
// ========== 组件映射 ==========
const components = {
    text: ElementText,
    barcode: ElementBarcode,
    qrcode: ElementQrcode,
    line: ElementLine,
    vline: ElementVLine,
    rectangle: ElementRectangle,
    table: ElementTable // 添加表格组件
};
// 键盘微调元素
const handleKeyDown = (e: KeyboardEvent) => {
    if (!props.currentElement) return;

    const step = 1; // 微调步长
    const { id, pos } = props.currentElement;
    const newTop = pos.top;
    const newLeft = pos.left;

    switch (e.key) {
        case 'ArrowUp':
            emits('update-element', { id, pos: { top: newTop - step, left: newLeft } });
            break;
        case 'ArrowDown':
            emits('update-element', { id, pos: { top: newTop + step, left: newLeft } });
            break;
        case 'ArrowLeft':
            emits('update-element', { id, pos: { top: newTop, left: newLeft - step } });
            break;
        case 'ArrowRight':
            emits('update-element', { id, pos: { top: newTop, left: newLeft + step } });
            break;
    }
};

onMounted(() => {
    window.addEventListener('keydown', handleKeyDown);
});

onUnmounted(() => {
    window.removeEventListener('keydown', handleKeyDown);
});
const getComponent = (type: string) => {
    return components[type] || ElementText
};

// ========== 拖动逻辑 ==========
interface DragState {
    isDragging: boolean;
    startX: number;
    startY: number;
    initialPos: Position;
}

const dragState = ref<DragState>({
    isDragging: false,
    startX: 0,
    startY: 0,
    initialPos: { top: 0, left: 0 }
});
const initialEl = ref({})
const startDrag = (el: ElementBase, e: MouseEvent) => {
    emits('select-element', el);
    dragState.value = {
        isDragging: true,
        startX: e.clientX,
        startY: e.clientY,
        initialPos: { ...el.pos,},
    };
    initialEl.value = { ...el }
    document.addEventListener('mousemove', onDrag);
    document.addEventListener('mouseup', endDrag);
};

const onDrag = (e: MouseEvent) => {
    if (!dragState.value.isDragging) return;
    const deltaX = e.clientX - dragState.value.startX;
    const deltaY = e.clientY - dragState.value.startY;
    const el = initialEl.value; // 获取初始元素
    emits('update-element', {
        id: el.id,
        pos: {
            top: dragState.value.initialPos.top + deltaY,
            left: dragState.value.initialPos.left + deltaX
        }
    });
};

const endDrag = () => {
    dragState.value.isDragging = false;
    document.removeEventListener('mousemove', onDrag);
    document.removeEventListener('mouseup', endDrag);
};

// ========== 缩放逻辑 ==========
interface ResizeState {
    isResizing: boolean;
    startX: number;
    startY: number;
    initialSize: Size;
    id: string; 
}

const resizeState = ref<ResizeState>({
    isResizing: false,
    startX: 0,
    startY: 0,
    initialSize: { width: 0, height: 0 },
    id: ''
});
const handleSize = ref(8); 
//样式方法
const handleStyle = (el:ElementBase,direction: string) => {
    const size = el.size;
    const handleSizeVal = handleSize.value;
    let style: any = { width: handleSizeVal + 'px', height: handleSizeVal + 'px' };
    // 如果元素有旋转，则添加旋转变换
    const rotation = el.style?.rotation || 0;
    if (rotation !== 0) {
        style.transform = `rotate(${rotation}deg)`;
        style.transformOrigin = 'center center';
    }
    switch (direction) {
        case 'nw':
            style.top = '-4px';
            style.left = '-4px';
            break;
        case 'n':
            style.top = '-4px';
            style.left = (size.width - handleSizeVal) / 2 + 'px';
            break;
        case 'ne':
            style.top = '-4px';
            style.right = '-4px';
            break;
        case 'e':
            style.top = (size.height - handleSizeVal) / 2 + 'px';
            style.right = '-4px';
            break;
        case 'se':
            style.bottom = '-4px';
            style.right = '-4px';
            break;
        case 's':
            style.bottom = '-4px';
            style.left = (size.width - handleSizeVal) / 2 + 'px';
            break;
        case 'sw':
            style.bottom = '-4px';
            style.left = '-4px';
            break;
        case 'w':
            style.top = (size.height - handleSizeVal) / 2 + 'px';
            style.left = '-4px';
            break;
    }

    return style;
};
const getSelectionStyle = (el: ElementBase) => {
    const baseStyle = {
        position: 'absolute' as const,
        top: '0px',
        left: '0px',
        width: `${el.size.width}px`,
        height: `${el.size.height}px`,
        pointerEvents: 'none' as const,
        zIndex: 9
    };

    // 如果是线条元素，添加边框和阴影样式
    if (el.type === 'line' || el.type === 'vline') {
        Object.assign(baseStyle, {
            border: '1px solid #1890ff',
            boxShadow: '0 0 5px rgba(24, 144, 255, 0.5)',
        });
    }

    // 添加旋转样式
    const rotation = el.style?.rotation || 0;
    if (rotation !== 0) {
        Object.assign(baseStyle, {
            transform: `rotate(${rotation}deg)`,
            transformOrigin: 'center center'
        });
    }

    return baseStyle;
};
const selectedElementStyle = (el: ElementBase) => {
    return {
        border: '1px solid #1890ff',
        boxShadow: '0 0 5px rgba(24, 144, 255, 0.5)'
    };
};

const onResize = (e: MouseEvent) => {
    if (!resizeState.value.isResizing) return;

    const dx = e.clientX - resizeState.value.startX;
    const dy = e.clientY - resizeState.value.startY;

    let newWidth = resizeState.value.initialSize.width;
    let newHeight = resizeState.value.initialSize.height;
    let newTop = props.elements.find(el => el.id === resizeState.value.id)?.pos.top || 0;
    let newLeft = props.elements.find(el => el.id === resizeState.value.id)?.pos.left || 0;

    switch (resizeDirection.value) {
        case 'nw':
            newWidth = resizeState.value.initialSize.width - dx;
            newHeight = resizeState.value.initialSize.height - dy;
            newTop += dy;
            newLeft += dx;
            break;
        case 'n':
            newHeight = resizeState.value.initialSize.height - dy;
            newTop += dy;
            break;
        case 'ne':
            newWidth = resizeState.value.initialSize.width + dx;
            newHeight = resizeState.value.initialSize.height - dy;
            newTop += dy;
            break;
        case 'e':
            newWidth = resizeState.value.initialSize.width + dx;
            break;
        case 'se':
            newWidth = resizeState.value.initialSize.width + dx;
            newHeight = resizeState.value.initialSize.height + dy;
            break;
        case 's':
            newHeight = resizeState.value.initialSize.height + dy;
            break;
        case 'sw':
            newWidth = resizeState.value.initialSize.width - dx;
            newHeight = resizeState.value.initialSize.height + dy;
            newLeft += dx;
            break;
        case 'w':
            newWidth = resizeState.value.initialSize.width - dx;
            newLeft += dx;
            break;
    }

    // 限制最小尺寸
    newWidth = Math.max(newWidth, 20);
    newHeight = Math.max(newHeight, 20);

    // 根据元素类型处理缩放逻辑
    const element = props.elements.find(el => el.id === resizeState.value.id);
    if (element) {
        switch (element.type) {
            case 'text':
                emits('update-element', {
                    id: element.id,
                    style: {
                        ...element.style,
                        fontSize: newHeight
                    },
                    size: {
                        ...element.size,
                        width: newWidth 
                    }
                });
                break;
            case 'barcode':
            case 'qrcode':
                // 等比缩放
                const scale = Math.min(newWidth / resizeState.value.initialSize.width, newHeight / resizeState.value.initialSize.height);
                newWidth = resizeState.value.initialSize.width * scale;
                newHeight = resizeState.value.initialSize.height * scale;
                emits('update-element', {
                    id: element.id,
                    size: { width: newWidth, height: newHeight },
                    pos: { top: newTop, left: newLeft }
                });
                break;
            case 'line':
            case 'vline':
                // 水平/垂直方向缩放
                if (resizeDirection.value.includes('e') || resizeDirection.value.includes('w')) {
                    newHeight = resizeState.value.initialSize.height;
                } else {
                    newWidth = resizeState.value.initialSize.width;
                }
                emits('update-element', {
                    id: element.id,
                    size: { width: newWidth, height: newHeight },
                    pos: { top: newTop, left: newLeft }
                });
                break;
            default:
                emits('update-element', {
                    id: element.id,
                    size: { width: newWidth, height: newHeight },
                    pos: { top: newTop, left: newLeft }
                });
        }
    }
};
const startResize = (el: ElementBase, direction: string, e: MouseEvent) => {
    e.stopPropagation();
    emits('select-element', el);
    resizeState.value = {
        isResizing: true,
        startX: e.clientX,
        startY: e.clientY,
        initialSize: { ...el.size },
        id: el.id
    };
    resizeDirection.value = direction;
    document.addEventListener('mousemove', onResize);
    document.addEventListener('mouseup', endResize);
};
const endResize = () => {
    resizeState.value.isResizing = false;
    document.removeEventListener('mousemove', onResize);
    document.removeEventListener('mouseup', endResize);
};

// ========== 选择逻辑 ==========
const selectElement = async (el: ElementBase) => {
    emits('select-element', el);
    await nextTick(); 
};

// ========== 默认值配置 ==========

const handleCanvasClick = (e: MouseEvent) => {
    if (e.target === e.currentTarget) {
        addElementByTool(e);
        emits('select-element', null); // 取消选中
    }
};
const addElementByTool = (e: MouseEvent) => {
    if (!props.currentTool || !isAddingMode.value) return;
    e.stopPropagation(); // 阻止冒泡
    //新增元素位置为画布上的点击位置
    const canvasRect = (e.currentTarget as HTMLElement).getBoundingClientRect();
    const x = e.clientX - canvasRect.left;
    const y = e.clientY - canvasRect.top;

    const newElement: ElementBase = {
        id: `${props.currentTool}-${Date.now()}`,
        type: props.currentTool,
        pos: { top: y, left: x },
        size: getDefaultSize(props.currentTool),
        content: getDefaultContent(props.currentTool),
        style: getDefaultStyle(props.currentTool),
        visible: true
    };
    emits('add-element', newElement);
    emits('select-element', newElement);

    isAddingMode.value = false; // 新增后退出添加模式
};
const getDefaultSize = (tool: string): Size => {
    switch (tool) {
        case 'text':
            return { width: 200, height: 30 };
        case 'barcode':
            return { width: 180, height: 100 };
        case 'line':
            return { width: 300, height: 3 }; 
        case 'vline':
            return { width: 300, height: 3 };
        case 'rectangle':
            return { width: 100, height: 50 };
        case 'table':
            return { width: 300, height: 150 };
        default:
            return { width: 100, height: 100 };
    }
};
// const getDefaultTableConfig = () => {
//     return {
//         rows: 3,
//         cols: 3,
//         width: 300,
//         height: 150,
//         rowHeight: 50,
//         colWidth: 100
//     };
// };
const getDefaultContent = (tool: string): string => {
    switch (tool) {
        case 'text':
            return '新文本';
        case 'barcode':
            return '123456';
        case 'qrcode':  
            return '123456'; 
        default:
            return '123456';
    }
};
const getDefaultStyle = (tool: string): Record<string, any> => {
    if (tool === 'text') {
        return { fontSize: 14, color: '#000' };
    } else if (tool === 'line' || tool === 'vline') {
        const style: Record<string, any> = {
            border: '1px solid #000',
            background: '#000'
        };
        // 竖线默认旋转90度，横线0度
        if (tool === 'vline') {
            style.rotation = 90;
        } else {
            style.rotation = 0;
        }
        return style;
    } else if (tool === 'rectangle') {
        return { border: '1px solid #000', background: 'transparent' };
    }

    return { color: '#000' };
};
const setAddingMode = (value: boolean) => {
    isAddingMode.value = value;
};
watch(
    () => props.currentElement,
    (newVal) => {
    }
);
// 暴露给父组件调用
defineExpose({ setAddingMode });
</script>

<style scoped>
.canvas-area {
    flex: 1;
    padding: 20px;
    background-color: #f5f5f5;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 80%;
}

.label-container {
    width: v-bind("labelSettings.width + 'px'");
    height: v-bind("labelSettings.height + 'px'");
    position: relative;
    border: 1px solid #ccc;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.element {
    cursor: move;
    border: 1px solid transparent;
}

.resize-handle {
    position: absolute;
    bottom: -5px;
    right: -5px;
    width: 10px;
    height: 10px;
    border: 1px solid #1890ff;
    background-color: #1890ff;
    cursor: se-resize;
    z-index: 2;
}
.resize-handles {
    position: absolute;
    /* 移除 top 和 left，通过内联样式设置 */
}
/* s缩放样式 */
.handle {
    position: absolute;
    width: 8px;
    height: 8px;
    background-color: #000;
    z-index: 10;
    pointer-events: auto;
}

.handle:hover {
    background-color: #f00;
}

.handle-nw {
    top: -4px;
    left: -4px;
    cursor: nwse-resize;
}

.handle-n {
    top: -4px;
    left: 50%;
    transform: translateX(-50%);
    cursor: ns-resize;
}

.handle-ne {
    top: -4px;
    right: -4px;
    cursor: nesw-resize;
}

.handle-e {
    right: -4px;
    top: 50%;
    transform: translateY(-50%);
    cursor: ew-resize;
}

.handle-se {
    bottom: -4px;
    right: -4px;
    cursor: nwse-resize;
}

.handle-s {
    bottom: -4px;
    left: 50%;
    transform: translateX(-50%);
    cursor: ns-resize;
}

.handle-sw {
    bottom: -4px;
    left: -4px;
    cursor: nesw-resize;
}

.handle-w {
    left: -4px;
    top: 55%;
    transform: translateY(-50%);
    cursor: ew-resize;
}
/* .handle-rotate {
    position: absolute;
    top: -50px;
    right: -10px;
    cursor: grab;
    z-index: 11;
} */

/* 线条选择框样式 */
.line-selection-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
}

.line-handle {
    position: absolute;
    width: 12px;
    height: 12px;
    background-color: #fff;
    border: 2px solid #1890ff;
    border-radius: 50%;
    pointer-events: auto;
    z-index: 10;
    transform: translate(-50%, -50%);
    cursor: pointer;
}

.line-handle-start {
    top: 50%;
    left: 0;
}

.line-handle-end {
    top: 50%;
    left: 100%;
}

.handle-rotate {
    position: absolute;
    top: -30px;
    left: 50%;
    width: 12px;
    height: 12px;
    background-color: #fff;
    border: 2px solid #1890ff;
    border-radius: 50%;
    pointer-events: auto;
    z-index: 11;
    transform: translate(-50%, -50%);
    cursor: grab;
}

.handle-rotate:hover {
    background-color: #1890ff;
}

.handle-rotate::after {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    width: 2px;
    height: 30px;
    background-color: #1890ff;
    transform: translate(-50%, -100%);
}

</style>