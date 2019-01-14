
# 安装依赖
npm install

Node.js 常见问题 之 node-sass 安装失败
```
Node.js 常见问题 之 node-sass 安装失败
问题分析
安装 node-sass 依赖时，会从 github.com 上下载 .node 文件。

由于网络环境的问题，多半会超时失败。

解决方案
使用淘宝cnpm安装

npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install node-sass
```
   
# 建议不要用cnpm安装 会有各种诡异的bug 可以通过如下操作解决 npm 下载速度慢的问题。
npm install --registry=https://registry.npm.taobao.org

# 启动服务
npm run dev
```
浏览器访问 http://localhost:9528

## 发布
```bash
# 构建 测试环境
npm run build:sit

# 构建生产环境
npm run build:prod
```

## 其它
```bash
# --report to build with bundle size analytics
npm run build:prod --report

# --preview to start a server in local to preview
npm run build:prod --preview

# lint code
npm run lint

# auto fix
npm run lint -- --fix
```
