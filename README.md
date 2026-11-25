# LeafOS Packager

基于 TurboWarp Packager 3.14.0 改编的 Scratch 作品打包器。支持将 SB3 项目打包为 HTML、ZIP、Windows EXE、Android APK。

## 项目结构

- `src/`：打包器与运行时源码
- `unpacker/`：本地解包工具
- `.github/workflows/`：GitHub Actions 构建、测试与 Pages 部署

## 在线工具

- 打包器：部署完成后访问站点首页
- 解包器：`/unpacker/`

解包器可尝试从本项目生成的 HTML 或 ZIP 中还原 SB3 项目。文件在浏览器本地处理，不会上传服务器。详细格式分析见 [解包格式说明](docs/UNPACKER.md)。

## 本地构建

需要 Node.js 22 或更高版本。

```bash
npm ci
npm run build-prod
npm run build-standalone-prod
```

## GitHub Pages

仓库使用 GitHub Actions 部署，触发分支为 `main`。请在仓库 Settings → Pages → Build and deployment 中将 Source 设为 GitHub Actions。

## 来源与许可

本项目基于 TurboWarp Packager 3.14.0 改编。请保留原项目的 MPL-2.0 许可文件、版权声明及适用的第三方许可。Scratch 名称与标志归其各自权利人所有。本项目与 TurboWarp、Scratch 或 MIT 无隶属关系，也未获其背书。
