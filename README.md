## 项目介绍
基于Vue3、Arco Design组件库等技术栈实现HXOJ在线判题系统前端设计，
能够根据管理员设定的题目测试用例在自主实现的代码沙箱中对代码进行编译、运行、判断输出是否正确。
其中，代码沙箱可以作为独立服务，提供给其他开发者使用。

### 技术选型
- Web 网页开发
- Vue 3
- Vue-CLI 脚手架
- Axios 请求库
- Arco Design 组件库
- 前端工程化：ESLint + Prettier + TypeScript
- 富文本编辑器
- ⭐️ OpenAPI 前端代码生成

## 项目安装和部署
### 1、安装项目依赖
```bash
npm install
# 
yarn 
```
本项目使用到的依赖/插件（不需执行）
```bash
# 组件库
yarn add --dev @arco-design/web-vue 
# Axios 工具
yarn add axios
# MarkDown 编辑器依赖
yarn add @bytemd/vue-next
# 为了使用多语言 locals目录在 bytemd 下
yarn add bytemd  
yarn add @bytemd/plugin-math @bytemd/plugin-highlight @bytemd/plugin-gfm 
yarn add highlight.js # 代码高亮

# 代码编辑器
yarn add monaco-editor
# 将webpack和文本编辑器整合在一起，便于打包和安装
yarn add monaco-editor-webpack-plugin 
```

### 获取后端代码接口
```
openapi --input http://localhost:8121/api//v2/api-docs --output ./backapi --client axios
```

### 2、项目构建
```bash
yarn build
```

### 3、生产环境运行
```
yarn serve
```

### 修复文件
```
yarn lint
```
