<template>
    <div class="element-barcode" :style="{ width: '100%', height: '100%' }">
        <canvas ref="barcodeRef" />
    </div>
</template>

<script setup lang="ts">
import { ref, defineProps, watch, onMounted } from 'vue';
import JsBarcode from 'jsbarcode';

// 接收父组件传递的element对象（包含内容、样式、大小等）
const props = defineProps({
    element: {
        type: Object,
        required: true
    }
});

// 条码 canvas 引用
const barcodeRef = ref<HTMLCanvasElement | null>(null);

// 生成条码的核心函数
const generateBarcode = () => {
    if (!barcodeRef.value || !props.element.content) return;

    // 从element.style中获取条码样式配置（默认值兜底）
    const {
        barcodeFormat = 'CODE128', // 条码类型（CODE128、EAN13等）
        barcodeWidth = 2,           // 条码线条宽度（px）
        barcodeHeight = 50,         // 条码高度（px）
        lineColor = '#000000',      // 条码颜色
        background = '#ffffff'      // 条码背景色
    } = props.element.style;

    // 使用JsBarcode生成条码
    JsBarcode(barcodeRef.value, props.element.content, {
        format: barcodeFormat,
        width: barcodeWidth,
        height: barcodeHeight,
        lineColor: lineColor,
        background: background,
        displayValue: true, // 是否显示条码下方的文本
        textPosition: 'bottom', // 文本位置（bottom/top）
        textMargin: 5 // 文本与条码的间距（px）
    });
};

// 监听element对象变化（内容或样式修改时重新生成条码）
watch(
    () => props.element,
    (newElement) => generateBarcode(),
    { deep: true } // 深度监听（因为element.style是对象）
);

// 组件挂载后生成初始条码
onMounted(() => generateBarcode());
</script>

<style scoped>
.element-barcode {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
}
</style>