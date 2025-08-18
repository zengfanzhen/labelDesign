<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
# labelDesign
标签编辑器
=======
# Vue 3 + TypeScript + Vite
=======
# 标签设计器 (Label Designer)
>>>>>>> 1b1838d (feat:标签编辑器README文档编写)

标签设计器是一个基于 Vue 3 + TypeScript + Vite 构建的可视化标签设计工具，支持创建包含文本、条码、二维码、线条、表格等元素的标签，并提供预览和打印功能。

<<<<<<< HEAD
Learn more about the recommended Project Setup and IDE Support in the [Vue Docs TypeScript Guide](https://vuejs.org/guide/typescript/overview.html#project-setup).
>>>>>>> 1ae1162 (feat:初始提交)
=======
=======
# 标签设计器 (Label Designer)

标签设计器是一个基于 Vue 3 + TypeScript + Vite 构建的可视化标签设计工具，支持创建包含文本、条码、二维码、线条、表格等元素的标签，并提供预览和打印功能。

>>>>>>> 3189268c0d336855a04a8e6e7e00b02eac433f64
## 功能特性

- **可视化编辑**: 拖拽式标签设计界面

- 多种元素支持:
  

  - 文本元素 (支持变量绑定)
  - 条形码 (CODE128)
  - 二维码 (QR Code)
  - 横线/竖线
  - 矩形
  - 表格 (支持合并单元格、调整行列大小)

- 元素操作:

  - 拖拽移动元素
  - 缩放调整元素大小
  - 键盘方向键微调位置
  - 复制/粘贴元素 (Ctrl+C/V)
  - 删除元素 (Delete键)

- 标签设置:

  - 自定义画布尺寸
  - 元素层级管理

- 数据绑定:

  - 文本元素支持绑定预定义变量
  - 预览时可输入变量值查看效果

- 导出功能:

  - 导出为 JSON 模板文件
  - 导入 JSON 模板文件
  - 生成标签预览图片
  - 打印功能

## 项目结构

```
label-designer/
├── src/
│   ├── components/
│   │   ├── CanvasArea.vue        # 画布区域组件
│   │   ├── Toolbar.vue           # 工具栏组件
│   │   ├── SettingsPanel.vue     # 设置面板组件
│   │   ├── LayersPanel.vue       # 图层面板组件
│   │   ├── Preview.vue           # 预览和导出组件
│   │   ├── Element*.vue          # 各类元素组件 (文本、条码、二维码等)
│   │   └── *Settings.vue         # 各类元素设置组件
│   ├── App.vue                   # 主应用组件
│   └── main.ts                   # 应用入口文件
├── public/
└── package.json
```

## 核心组件说明

### 1. App.vue

主应用组件，整合所有功能模块，包括:

- 工具栏 (Toolbar)
- 画布区域 (CanvasArea)
- 设置面板 (SettingsPanel)
- 预览功能 (Preview)

### 2. CanvasArea.vue

标签设计的核心画布区域:

- 显示标签背景和所有元素
- 支持元素的添加、选择、拖拽、缩放
- 处理元素间的层级关系

### 3. Toolbar.vue

左侧工具栏，提供添加各类元素的功能:

- 文本、条码、二维码
- 横线、竖线、矩形
- 表格 (带配置对话框)

### 4. SettingsPanel.vue

右侧设置面板，根据选中元素类型显示对应设置:

- 标签画布设置
- 文本元素设置 (内容、字体、颜色等)
- 条码/二维码设置
- 线条设置 (颜色、粗细等)
- 表格设置 (行列数、边框等)

### 5. Preview.vue

预览和导出功能:

- 标签预览 (支持变量替换)
- 打印功能
- 导出为 JSON 模板
- 导入 JSON 模板

### 6. 元素组件

各类标签元素的具体实现:

- ElementText.vue: 文本元素
- ElementBarcode.vue: 条码元素
- ElementQrcode.vue: 二维码元素
- ElementLine.vue/ElementVLine.vue: 横线/竖线元素
- ElementRectangle.vue: 矩形元素
- ElementTable.vue: 表格元素

## 使用说明

### 基本操作流程

1. **设置画布尺寸**: 在右侧设置面板中调整标签的宽度和高度

2. 添加元素:

   - 从左侧工具栏选择需要的元素类型
   - 在画布上点击添加(文本、条码等)或通过配置对话框添加(表格)

3. 编辑元素:

   - 点击选中元素，在右侧设置面板中调整属性
   - 拖拽移动元素位置
   - 使用缩放手柄调整元素大小
   - 键盘方向键微调元素位置

4. 绑定变量(文本元素):

   - 选中文本元素
   - 在设置面板中选择绑定变量

5. 预览和打印:

   - 点击"预览"按钮查看效果
   - 点击"预览绑定变量"可输入变量值查看实际效果
   - 点击"打印"按钮打印标签

6. 保存和加载:

   - 使用"导出为json"保存模板
   - 使用"导入json文件"加载模板

### 快捷键

- `Ctrl + C`: 复制选中元素
- `Ctrl + V`: 粘贴元素
- `Delete`: 删除选中元素
- 方向键: 微调选中元素位置

## 技术栈

- Vue 3 (Composition API)
- TypeScript
- Vite
- Element Plus (UI 组件库)
- JsBarcode (条形码生成)
- QRCode (二维码生成)
- html2canvas (截图功能)
- Lodash (工具函数)

## 开发指南

### 安装依赖

```
npm install
```

### 启动开发服务器

```
npm run dev
```

### 构建生产版本

```
npm run build
```

### 代码检查

```
npm run lint
```

## 扩展功能

项目具有良好的扩展性，可以轻松添加新的元素类型:

1. 创建新的元素组件 (如 `Element*.vue`)
2. 创建对应的设置组件 (如 `*Settings.vue`)
3. 在 `CanvasArea.vue` 中注册组件
4. 在 `SettingsPanel.vue` 中添加组件映射
5. 在 `Toolbar.vue` 中添加工具按钮

## 注意事项

1. 表格元素功能较为复杂，支持合并单元格和调整行列大小
2. 文本元素支持双击编辑内容
3. 预览功能支持变量替换，可查看实际打印效果
<<<<<<< HEAD
4. 导出的 JSON 文件包含完整的标签模板信息，可跨设备使用
>>>>>>> 1b1838d (feat:标签编辑器README文档编写)
=======
4. 导出的 JSON 文件包含完整的标签模板信息，可跨设备使用
>>>>>>> 3189268c0d336855a04a8e6e7e00b02eac433f64
