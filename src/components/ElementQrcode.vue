<template>
    <div class="element-qrcode" :style="{ width: '100%', height: '100%' }">
        <canvas ref="qrcodeRef" />
    </div>
</template>

<script setup lang="ts">
import { ref, defineProps, watch, onMounted } from 'vue';
import QRCode from 'qrcode';

// 接收父组件传递的element对象
const props = defineProps({
    element: {
        type: Object,
        required: true
    }
});

// 二维码 canvas 引用
const qrcodeRef = ref<HTMLCanvasElement | null>(null);

// 生成二维码的核心函数
const generateQrcode = async () => {
    if (!qrcodeRef.value || !props.element.content) return;

    try {
        // 设置canvas尺寸
        qrcodeRef.value.width = props.element.size.width;
        qrcodeRef.value.height = props.element.size.height;

        // 从element.style中获取二维码样式配置（默认值兜底）
        const {
            qrcodeSize = Math.min(props.element.size.width, props.element.size.height), // 使用元素尺寸
            colorDark = '#000000',    // 二维码前景色
            colorLight = '#ffffff',   // 二维码背景色
            // correctLevel = QRCode.CorrectLevel.H // 容错级别（L/M/H/Q）
        } = props.element.style || {};
        console.log('QRCode:', QRCode);
        // 使用qrcode直接绘制到canvas
        await QRCode.toCanvas(qrcodeRef.value, props.element.content, {
            width: qrcodeSize,
            color: {
                dark: colorDark,
                light: colorLight
            },
            // correctLevel: correctLevel,
            margin: 1
        });
    } catch (error) {
        console.error('生成二维码失败:', error);
    }
};

// 监听element对象变化（内容或样式修改时重新生成二维码）
watch(
    () => props.element,
    (newElement) => generateQrcode(),
    { deep: true }
);

// 组件挂载后生成初始二维码
onMounted(() => generateQrcode());
</script>

<style scoped>
.element-qrcode {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
}
</style>