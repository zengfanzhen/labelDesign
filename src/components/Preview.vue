<template>
    <div class="preview-container">
        <!-- 操作按钮 -->
        <div class="preview-actions">
            <el-button @click="showPreview">
                <el-icon>
                    <View />
                </el-icon>
                预览
            </el-button>
            <el-button @click="showPreviewTest">
                <el-icon>
                    <View />
                </el-icon>
                预览绑定变量
            </el-button>
            <el-button type="primary" @click="print">
                <el-icon>
                    <Stamp />
                </el-icon>
                打印
            </el-button>
            <el-button @click="exportToJson" type="primary">
                <el-icon>
                    <Download />
                </el-icon>
                导出为json
            </el-button>
            <el-button @click="triggerFileImport" type="primary">
                <el-icon>
                    <Upload />
                </el-icon>
                导入json文件
            </el-button>
            <input ref="fileInput" type="file" accept=".json" @change="handleFileImport" style="display: none" />
        </div>
        <!-- 变量输入弹窗 -->
        <el-dialog v-model="variableInputDialogVisible" title="输入变量值" width="500px"
            :before-close="handleVariableInputDialogClose">
            <el-form>
                <el-form-item v-for="variable in usedVariables" :key="variable" :label="getVariableLabel(variable)">
                    <el-input v-model="variableInputs[variable]" :placeholder="`请输入${variable}的值`" />
                </el-form-item>
            </el-form>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="cancelPreview">取消</el-button>
                    <el-button type="primary" @click="confirmVariables">确认</el-button>
                </span>
            </template>
        </el-dialog>
        <!-- 预览弹窗 -->
        <el-dialog v-model="previewVisible" title="打印标签预览" width="70%" top="30px" :before-close="handlePreviewClose">
            <div class="preview-content">
                <img v-if="previewImage" :src="previewImage" alt="标签预览" class="preview-image" />
                <div v-else class="loading">生成预览中...</div>
            </div>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="previewVisible = false">关闭</el-button>
                </span>
            </template>
        </el-dialog>
    </div>
</template>

<script setup lang="ts">
import { defineProps, defineEmits, ref, computed ,reactive } from 'vue';
import { Stamp, View, Download, Upload } from '@element-plus/icons-vue';
import { ElMessage } from 'element-plus';

const props = defineProps({
    labelSettings: Object,
    elements: Array,
    dataVariables: Object
});

const emits = defineEmits(['close', 'import-data']);
// 预览弹窗可见性
const previewVisible = ref(false);
// 预览图片数据
const previewImage = ref<string | null>(null);
// 文件输入引用
const fileInput = ref<HTMLInputElement | null>(null);
// 计算画布上使用的变量
const usedVariables = computed(() => {
    const variables = new Set<string>();
    console.log('props.elements', props.elements);
    if (props.elements) {
        props.elements.forEach((element: any, index: number) => {
            console.log('element', element);
            // 确保 content 存在且为字符串
            if (element.content && typeof element.content === 'string') {
                // 同时匹配 {{variableName}} 和 ${variableName} 格式的变量
                const regex1 = /{{(.*?)}}/g;  // {{variableName}} 格式
                const regex2 = /\${(.*?)}/g;  // ${variableName} 格式
                let match;
                let matchCount = 0;

                // 匹配 {{variableName}} 格式
                while ((match = regex1.exec(element.content)) !== null) {
                    variables.add(match[1]);
                    matchCount++;
                }

                // 匹配 ${variableName} 格式
                while ((match = regex2.exec(element.content)) !== null) {
                    variables.add(match[1]);
                    matchCount++;
                }
            }
        });
    }

    const result = Array.from(variables);
    console.log('Final variables list:', result);
    return result;
});
// 变量输入值存储
const variableInputs = reactive<Record<string, string>>({});
// 变量输入弹窗可见性
const variableInputDialogVisible = ref(false);

// 关闭变量输入对话框
const handleVariableInputDialogClose = () => {
    variableInputDialogVisible.value = false;
};
// 添加获取变量标签的方法
const getVariableLabel = (variableKey: string) => {
    // 首先在elements中查找匹配的变量元素，获取selectedVariableLabel
    if (props.elements) {
        for (const element of props.elements) {
            // 检查元素是否包含变量且内容匹配
            if (element.content && typeof element.content === 'string') {
                // 检查是否是${variableKey}格式
                if (element.content === `\${${variableKey}}`) {
                    // 如果元素有selectedVariableLabel属性且不为空，则使用它
                    if (element.selectedVariableLabel) {
                        return element.selectedVariableLabel;
                    }
                }
            }
        }
    }
    return variableKey;
};
// 用变量值生成预览
const generatePreviewWithVariables = (variables: Record<string, string>) => {
    previewVisible.value = true;
    previewImage.value = null;

    // 等待弹窗打开后生成预览
    setTimeout(() => {
        generatePreviewTest(variables);
    }, 100);
};
//绑定变量
const showPreviewTest = () => {
    // 检查是否有变量需要输入
    if (usedVariables.value.length > 0) {
        // 初始化变量输入值
        usedVariables.value.forEach(variable => {
            // 如果dataVariables中有默认值，则使用默认值，否则为空
            variableInputs[variable] = (props.dataVariables as Record<string, string>)?.[variable] || '';
        });

        // isPrintOperation.value = false;
        variableInputDialogVisible.value = true;
    } else {
        // 没有变量直接生成预览
        generatePreviewWithVariables({});
    }
};
// 确认变量输入
const confirmVariables = () => {
    variableInputDialogVisible.value = false;
    generatePreviewWithVariables({ ...variableInputs });

    // 根据操作类型执行相应功能
    // if (isPrintOperation.value) {
    //     executePrintWithVariables({ ...variableInputs });
    // } else {
    // }
};

// 取消预览
const cancelPreview = () => {
    variableInputDialogVisible.value = false;
    previewVisible.value = false;
};
// 显示预览弹窗
const showPreview = () => {
    previewVisible.value = true;
    previewImage.value = null;
    // 等待弹窗打开后生成预览
    setTimeout(() => {
        generatePreview();
    }, 100);
};
// 创建预览容器
const createPreviewContainer = (variables: Record<string, string>) => {
    // 获取现有的画布容器
    const originalCanvas = document.querySelector('.canvas-area .label-container');
    if (!originalCanvas) {
        throw new Error('无法找到画布容器');
    }
    
    // 克隆画布容器及其所有子元素
    const clonedCanvas = originalCanvas.cloneNode(true) as HTMLElement;

    // 设置克隆容器的样式
    clonedCanvas.style.position = 'absolute';
    clonedCanvas.style.left = '-9999px';
    clonedCanvas.style.top = '-9999px';
    clonedCanvas.style.width = (props.labelSettings?.width || 800) + 'px';
    clonedCanvas.style.height = (props.labelSettings?.height || 600) + 'px';
    clonedCanvas.style.boxShadow = '0 0 10px rgba(0, 0, 0, 0.1)';

    // 处理所有元素中的变量替换
    const allElements = clonedCanvas.querySelectorAll('.element');
    allElements.forEach((elementDiv, index) => {
        // 找到对应的原始元素
        const originalElement = props.elements?.[index];
        if (!originalElement || !originalElement.content) return;

        let content = originalElement.content;
        let hasVariables = false;

        // 检查是否包含变量
        Object.keys(variables).forEach(key => {
            const regex1 = new RegExp(`{{${key}}}`, 'g');
            const regex2 = new RegExp(`\\\${${key}}`, 'g');
            if (regex1.test(content) || regex2.test(content)) {
                hasVariables = true;
            }
        });

        // 如果包含变量，则进行替换
        if (hasVariables) {
            Object.keys(variables).forEach(key => {
                // 替换 {{variable}} 格式
                const regex1 = new RegExp(`{{${key}}}`, 'g');
                content = content.replace(regex1, variables[key]);

                // 替换 ${variable} 格式
                const regex2 = new RegExp(`\\\${${key}}`, 'g');
                content = content.replace(regex2, variables[key]);
            });

            // 根据元素类型更新内容
            switch (originalElement.type) {
                case 'text':
                    const textElement = elementDiv.querySelector('.element-text');
                    if (textElement) {
                        textElement.textContent = content;
                    }
                    break;
                // case 'barcode':
                // case 'qrcode':
                //     // 对于条形码和二维码，我们需要更新其子组件中的内容
                //     // 这里假设条形码和二维码组件内部有文本显示
                //     const codeElement = elementDiv.firstChild as HTMLElement;
                //     if (codeElement) {
                //         codeElement.textContent = content;
                //     }
                //     break;

                default:
                    // 其他元素类型保持原样
                    break;
            }
        }
    });
    // 将克隆的容器添加到body中
    document.body.appendChild(clonedCanvas);
    return clonedCanvas;
};
// 生成预览图片
const generatePreviewTest = (variables: Record<string, string>) => {
    console.log('variables',variables);
    // 创建临时容器用于生成预览
    const tempContainer = createPreviewContainer(variables);
    document.body.appendChild(tempContainer);

    // 使用html2canvas库截图CanvasArea
    import('html2canvas').then(html2canvas => {
        html2canvas.default(tempContainer, {
            backgroundColor: '#ffffff',
            scale: 2, // 适当提高截图质量
            useCORS: true,
        }).then(canvas => {
            // 将canvas转换为图片
            previewImage.value = canvas.toDataURL('image/png');
            // 移除临时容器
            document.body.removeChild(tempContainer);
        }).catch(err => {
            console.error('生成预览失败:', err);
            ElMessage.error('生成预览失败');
            // 移除临时容器
            document.body.removeChild(tempContainer);
        });
    });
};

// 生成预览图片
const generatePreview = () => {
    // 获取CanvasArea元素
    const canvasArea = document.querySelector('.canvas-area .label-container');
    if (canvasArea) {
        // 使用html2canvas库截图CanvasArea
        import('html2canvas').then(html2canvas => {
            html2canvas.default(canvasArea as HTMLElement, {
                backgroundColor: '#ffffff',
                scale: 2, // 适当提高截图质量
                useCORS: true,
            }).then(canvas => {
                // 将canvas转换为图片
                previewImage.value = canvas.toDataURL('image/png');
            });
        });
    }
};

// 关闭预览弹窗
const handlePreviewClose = () => {
    previewVisible.value = false;
    previewImage.value = null;
};
// 导出为 JSON 文件
const exportToJson = () => {
    // 将 elements 和 labelSettings 组合成一个对象
    const exportData = {
        labelSettings: props.labelSettings,
        elements: props.elements,
        dataVariables: props.dataVariables
    };
    // 转换为 JSON 字符串
    const jsonData = JSON.stringify(exportData, null, 2);
    // 创建 Blob 对象
    const blob = new Blob([jsonData], { type: 'application/json' });
    // 创建下载链接
    const url = URL.createObjectURL(blob);
    console.log(url, 'url');
    const a = document.createElement('a');
    a.href = url;
    const now = new Date();
    const timestamp = `${now.getFullYear()}${(now.getMonth() + 1).toString().padStart(2, '0')}${now.getDate().toString().padStart(2, '0')}_${now.getHours().toString().padStart(2, '0')}${now.getMinutes().toString().padStart(2, '0')}${now.getSeconds().toString().padStart(2, '0')}`;
    a.download = `label-printing-template-${timestamp}.json`;
    document.body.appendChild(a);
    a.click();
    // 清理
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
};
// 触发文件选择
const triggerFileImport = () => {
    if (fileInput.value) {
        fileInput.value.click();
    }
};

// 处理文件导入
const handleFileImport = (event: Event) => {
    const target = event.target as HTMLInputElement;
    console.log(target, 'target');
    const file = target.files?.[0];
    console.log(file, 'file');
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (e) => {
        try {
            const content = e.target?.result as string;
            const importedData = JSON.parse(content);
            console.log(content, 'content');
            // 验证数据格式
            if (!importedData.labelSettings || !importedData.elements) {
                console.log('11111');
            }
            // 发送数据到父组件
            emits('import-data', importedData);
            // 重置文件输入
            target.value = '';
        } catch (error) {
            ElMessage.error('导入文件失败，请检查文件格式是否正确');
        }
    };

    reader.readAsText(file);
};
// 打印 CanvasArea 画布内容
const print = () => {
    // 获取CanvasArea元素
    const canvasArea = document.querySelector('.canvas-area .label-container');
    console.log(canvasArea, 'canvasArea');
    if (canvasArea) {
        // 使用html2canvas库截图CanvasArea
        import('html2canvas').then(html2canvas => {
            html2canvas.default(canvasArea as HTMLElement, {
                backgroundColor: '#ffffff',
                scale: 10, // 提高截图质量
                useCORS: true,
            }).then(canvas => {
                // 将canvas转换为图片
                const dataUrl = canvas.toDataURL('image/png');
                // 创建打印窗口
                const printWindow = window.open('', '_blank');
                if (printWindow) {
                    // 构建打印内容
                    const printContent = `
                        <!DOCTYPE html>
                        <html>
                        <head>
                          <style>
                            body {
                              margin: 0;
                              padding: 0;
                              display: flex;
                              justify-content: center;
                              align-items: center;
                              min-height: 100vh;
                            }
                            .print-container {
                              text-align: center;
                            }
                            .label-image {
                              max-width: 100%;
                              height: auto;
                            }
                          </style>
                        </head>
                        <body>
                          <div class="print-container">
                            <img class="label-image" src="${dataUrl}" alt="标签" />
                          </div>
                        </body>
                        </html>
                    `;
                    // 写入打印内容
                    printWindow.document.write(printContent);
                    printWindow.document.close();

                    // 等待内容加载完成后打印
                    printWindow.onload = function () {
                        printWindow.focus();
                        printWindow.print();
                        printWindow.close();
                    };
                }
            });
        });
    }
};

</script>

<style scoped>
.preview-container {
    padding: 20px;
}

.preview-actions {
    margin-bottom: 20px;
    text-align: right;
}

.preview-content {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 60vh;
    max-height: 70vh;
    overflow: auto;
}

.preview-image {
    max-width: 100%;
    max-height: 70vh;
    object-fit: contain;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.loading {
    font-size: 16px;
    color: #666;
}

/* 打印样式（隐藏不必要的元素） */
@media print {
    .preview-actions {
        display: none;
    }

    .preview-container {
        padding: 0;
    }
}
</style>