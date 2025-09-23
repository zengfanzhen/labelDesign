<template>
  <div class="app-container">
    <el-container style="height: 100%;">
      <el-aside width="140px">
        <Toolbar @select-tool="setCurrentTool" />
      </el-aside>
      <el-container>
        <el-header style="height: 40px;">
          <Preview :label-settings="labelSettings" :elements="elements" :data-variables="dataVariables"
            @close="closePreview" @import-data="handleImportData" />
        </el-header>
        <el-container>
          <el-main>
            <!-- <CanvasArea ref="canvasAreaRef" :current-element="currentElement" :current-tool="currentTool"
              :label-settings="labelSettings" :elements="elements" @select-element="setCurrentElement"
              @update-element="updateElement" @add-element="addElement" /> -->
            <CanvasArea ref="canvasAreaRef" :current-element="currentElement" :current-tool="currentTool"
              :label-settings="labelSettings" :elements="elements" :data-variables="dataVariables"
              @select-element="setCurrentElement" @update-element="updateElement" @add-element="addElement" />
          </el-main>
          <el-aside width="260px" style="margin-right: 20px;">
            <SettingsPanel :current-element="currentElement" :label-settings="labelSettings" :elements="elements"
              @update-label="updateLabelSettings" @update-element="updateElement" @clear-canvas="clearCanvas" />
          </el-aside>
        </el-container>
      </el-container>
    </el-container>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';
import Toolbar from './components/Toolbar.vue';
import CanvasArea from './components/CanvasArea.vue';
import SettingsPanel from './components/SettingsPanel.vue';
import Preview from './components/Preview.vue';
import { cloneDeep } from 'lodash-es';
import { ElMessage } from 'element-plus'; 
import { backgroundClip } from 'html2canvas/dist/types/css/property-descriptors/background-clip';

// 元素选择
const setCurrentElement = (element: any) => {
  currentElement.value = element;
};

// 状态管理
const currentTool = ref<string | null>(null); // 当前选中的工具（文本/条码等）
const currentElement = ref<any | null>(null); // 当前选中的元素
const copiedElement = ref<any | null>(null); // 复制的元素
const labelSettings = reactive({ width: 1400, height: 800, overflow: true, background: '',backgroundColor: '#ffffff' }); // 标签全局设置
const elements = reactive<any[]>([]); // 元素列表（文本、条码等）
// const activeTab = ref('设置'); // 右侧面板当前 tab
const isPreviewOpen = ref(false); // 是否显示预览弹窗
//绑定数据变量（预留）
const dataVariables = reactive({ inspectionTaskNumber: '' }); 
const canvasAreaRef = ref();

//================处理导入数据=======================
const handleImportData = (importedData: any) => {
  try {
    // 清空当前画布
    clearCanvas();
    console.log('elements', elements);
    // 更新标签设置
    Object.assign(labelSettings, importedData.labelSettings);
    // 更新元素列表
    elements.push(...importedData.elements);
    console.log('elements1', elements);
    // 绑定数据变量
    if (importedData.dataVariables) {
      Object.assign(dataVariables, importedData.dataVariables);
    }
    // 清除当前选中元素
    currentElement.value = null;
    ElMessage.success('导入成功！');
  } catch (error) {
    ElMessage.error('导入失败，请检查文件格式是否正确');
  }
};
//================清空画布=======================

const clearCanvas = () => {
  console.log('elements', elements);
  elements.splice(0); // 清空元素列表
  currentElement.value = null; // 清除当前选中元素
};
//================删除元素=====================
// 删除元素
const deleteElement = (id: string) => {
  const index = elements.findIndex(el => el.id === id);
  if (index !== -1) elements.splice(index, 1);
};

//================复制粘贴功能=====================
// 复制选中的元素
const copyElement = () => {
  if (currentElement.value) {
    copiedElement.value = cloneDeep(currentElement.value);
    // copiedElement.value = currentElement.value;
  }
};

// 粘贴复制的元素
const pasteElement = () => {
  if (copiedElement.value) {
    const newElement = cloneDeep(copiedElement.value);
    // const newElement = copiedElement.value;//修改粘贴后的元素会影响原元素，引用同一个对象
    newElement.id = `element-${Date.now()}`; 
    console.log(newElement,'newElement');
    // 复制位置
    newElement.pos = {
      top: (newElement.pos?.top ?? 0) + 20,
      left: (newElement.pos?.left ?? 0) + 20
    };
    if (!newElement.size) {
      newElement.size = { width: 100, height: 30 };
    }
    // 添加元素
    addElement(newElement);
    // 选中新元素
    setCurrentElement(newElement); 
  }
};
onMounted(() => {
  window.addEventListener('keydown', handleKeyDown);
});

const handleKeyDown = (e: KeyboardEvent) => {
  if (e.ctrlKey || e.metaKey) {
    if (e.key === 'c') {
      copyElement(); // 复制
    } else if (e.key === 'v') {
      pasteElement(); // 粘贴
    }
  }
  // 添加删除功能
  else if (e.key === 'Delete') {
    if (currentElement.value) {
      deleteElement(currentElement.value.id);
      currentElement.value = null;
    }
  }
};
//=================工具功能=====================
// 工具选择
// const setCurrentTool = (tool: string) => {
//   currentTool.value = tool
//   canvasAreaRef.value?.setAddingMode(true); 
// };
const setCurrentTool = (tool: string, config?: any) => {
  if (tool === 'table') {
    // 对于表格，我们直接添加元素而不是设置当前工具
    addTableElement(config);
  } else {
    currentTool.value = tool;
    canvasAreaRef.value?.setAddingMode(true);
  }
};
//=================表格功能=====================
// 添加添加表格元素的函数
const addTableElement = (config: any) => {
  const tableConfig = config || getDefaultTableConfig();

  const newElement = {
    id: `table-${Date.now()}`,
    type: 'table',
    pos: { top: 100, left: 100 }, // 默认位置
    size: {
      width: tableConfig.width,
      height: tableConfig.height
    },
    rows: tableConfig.rows,
    cols: tableConfig.cols,
    width: tableConfig.width,
    height: tableConfig.height,
    rowHeight: tableConfig.rowHeight,
    colWidth: tableConfig.colWidth,
    style: {}
  };

  addElement(newElement);
  setCurrentElement(newElement);
};

// 添加获取默认表格配置的函数
const getDefaultTableConfig = () => {
  return {
    rows: 3,
    cols: 3,
    width: 300,
    height: 150,
    rowHeight: 50,
    colWidth: 100
  };
};


//=================元素操作功能=====================
// 添加元素
const addElement = (element: any) => elements.push(element);

// 更新元素
const updateElement = (updatedElement: any) => {
  const index = elements.findIndex(el => el.id === updatedElement.id);
  //更新元素数组中的元素
  if (index !== -1) {
    elements[index] = { ...elements[index], ...updatedElement };
  }
  //同步更新当前选中元素
  if (currentElement.value?.id === updatedElement.id) {
    currentElement.value = { ...currentElement.value, ...updatedElement };
  }
};

// 更新标签全局设置
const updateLabelSettings = (settings: any) => Object.assign(labelSettings, settings);

// 打开/关闭预览
const closePreview = () => isPreviewOpen.value = false;


</script>

<style scoped>
#app{
  margin: 0;
    padding: 0;
}
.app-container {
  margin: 0;
  padding: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  box-sizing: border-box;
}

.el-container,
.el-main,
.el-aside,
.el-header,
.el-footer {
  height: 100%;
}

.el-main {
  overflow-y: auto;
}
</style>