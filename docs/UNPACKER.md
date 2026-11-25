# 解包格式研究记录

## 结论

当前源码的“防解包”实现是可逆的轻量混淆，不是强加密。解包所需的密钥和解密算法都随输出文件一起交付给浏览器，因此能够编写兼容解包器。它可以提高随手查看数据的门槛，但不能阻止有技术能力的人还原项目。

## 源码依据

- `src/packager/protect.js` 定义 `makeKey`、`cryptBytes` 与 `encryptProject`。
- `src/packager/packager.js` 的 `generateGetProjectData()` 生成 HTML / ZIP 的项目数据加载代码。
- HTML 目标把项目数据切成 128 KiB 块，逐块进行 Base85 编码；启用保护时，每块先经过 `cryptBytes`。
- ZIP 类目标把项目字节写入 `project.dat`；启用保护时，写入前调用 `encryptProject`。同一 ZIP 内的 `index.html` 含有运行时所需的密钥。

## 算法摘要

1. 密钥是 4 个 32 位无符号整数。若用户填写自定义密钥，源码通过 FNV-1a 风格的字符串散列和 xorshift 扩展生成；否则使用浏览器的 `crypto.getRandomValues`。
2. 每个块的初始状态为 `key[0] XOR imul(chunkIndex + 1, 0x9e3779b1)`。若结果为零，则替换为 `0x6d2b79f5`。
3. 状态使用 xorshift32 更新：左移 13、无符号右移 17、左移 5。
4. 数据按小端序每 4 字节与伪随机状态异或；不足 4 字节的尾部按字节异或。
5. HTML 的 Base85 字符范围为 ASCII `0x2a..0x7e`，其中 `<` 和 `>` 分别替换为 `(` 和 `)`，避免破坏 HTML。

## 解包器支持范围

`unpacker/index.html` 实现了以下路径：

- HTML：读取内嵌的 `zxKey`，找到 `data` 属性和 `decodeChunk(size, index)`，Base85 解码后按块索引解密，拼回 SB3 字节流。
- ZIP：读取 `index.html` 中的密钥，提取 `project.dat`，使用块索引 0 解密。
- 未启用保护的输出：密钥为 `null` 时不做解密。

解包器依赖 jsDelivr 上的 JSZip 来读取 ZIP。若需要完全离线运行，应将 JSZip 文件一并托管在站点内。

## 限制与注意事项

- 目前只针对本仓库这套格式，不保证兼容原版 TurboWarp Packager 或其他分支。
- 目前解包器只处理 HTML 与 ZIP；不解析 Windows EXE、APK、Electron/NW.js 安装包或 macOS 应用。
- 若输出文件被二次压缩、改写脚本结构或改变密钥变量命名，HTML 自动识别可能失败。
- 自定义密钥不是独立的解包口令：当前输出 HTML / ZIP 必须把运行时解密所需密钥交给播放器，所以不能把它当成真正的保密机制。
- 解包后得到的是项目归档字节，不保证其内容一定是有效 SB3；工具会提供基本长度检查，但没有完整的 Scratch 项目校验器。

## 安全与授权

仅对自己拥有或获授权处理的项目使用。解包不改变原项目版权、素材授权或使用条款。该工具不会上传文件，但浏览器仍需联网加载 JSZip CDN。
