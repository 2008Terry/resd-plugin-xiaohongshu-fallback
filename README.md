# 小红书插件：独立修复版

这是 [putyy 的官方小红书插件](https://github.com/putyy/resd-plugin-xiaohongshu)的独立派生版本，基于 upstream commit `a762822a3f71e726708fb44043119d8c0b8cfcb1`。原项目及本派生版本使用 Apache-2.0 许可证；本仓库修改了 `main.js`、`plugin.json` 和 `tests/contract_test.go`。

**需要配套的 res 主程序修复。** 本插件只为图片下载计划设置 `xiaohongshuBackup`；它本身无法检查下载是否失败。默认 res 4.0.0-beta.7 不支持该选项，安装本插件并不能单独修复 404。配套主程序源码在 [res-downloader 修复分支](https://github.com/2008Terry/res-downloader-xhs-fallback/tree/fix/xhs-image-cdn-fallback)。该方案尚未被上游合并或发布。

修复后的主程序先使用原始图片地址；遇到 404、服务器错误或网络错误时，才使用备用 CDN。403、取消和本地文件错误不会触发切换。链接过期或两个地址均失败时仍需重新抓取。

插件 ID 为 `terry.xiaohongshu-fallback`，与官方插件分开。使用时请先关闭官方小红书插件以避免重复捕获；旧资源可能属于官方插件，需要重新抓取。构建插件包后通过 res 的「插件管理 → 从压缩包安装」安装。切勿将本插件误认为官方发布版本。独立插件不会被官方插件更新直接覆盖，但官方主程序更新可能覆盖主程序修复，更新后必须重新验证或重建。

验证：独立插件契约测试通过；配套主程序通过针对回退的测试。2026-10-28 使用本地修复版 res 下载一篇真实图集，15 张图片及文案完成；其中一张图片默认地址返回 404 后通过备用地址成功下载。此验证不保证所有作品或未来 CDN 地址始终可用。

打包可在配套 res 源码根目录运行 `res-downloader plugin pack <本插件目录>`，生成 `dist/plugin.zip`。仓库不提供未重新构建的旧安装包。

---

# 小红书

[中文](README.md) | [English](README-EN.md)

[res-downloader](https://github.com/putyy/res-downloader) 的小红书作品插件。

## 功能

- 支持打开作品后识别视频、单图、图集和 Live Photo。
- 视频优先下载最高可用画质，支持音视频合并。
- 支持按作品合集下载，也可单独下载图片、视频或 TXT 文案。

## 安装

发布后可在 `res-downloader` 的“插件管理”页面安装。也可以下载对应版本的源码 ZIP，通过“从压缩包安装”导入。

## 设置

- **启用日志**：默认关闭，用于排查抓取问题。

## 注意事项

- 分轨视频下载需要在应用中配置 FFmpeg。
- 视频预览画质可能低于下载画质，部分视频仅显示封面。
- Live Photo 保存为独立的图片和视频文件，不生成苹果相册的配对实况照片。
- 链接过期或文案文件丢失时，刷新页面并重新打开作品抓取。
- 不支持批量抓取首页推荐、字幕和 DRM 内容；空文案或超过 16384 字符的文案不生成 TXT。

## 开发与校验

在 `res-downloader` 项目根目录执行：

```bash
node --check ./plugins/resd-plugin-xiaohongshu/main.js
go run main.go plugin lint ./plugins/resd-plugin-xiaohongshu
for fixturePath in ./plugins/resd-plugin-xiaohongshu/fixtures/*.json; do
  go run main.go plugin replay ./plugins/resd-plugin-xiaohongshu "$fixturePath" || break
done
go test ./plugins/resd-plugin-xiaohongshu/tests
go run main.go plugin pack ./plugins/resd-plugin-xiaohongshu
```

安装包位于 `dist/plugin.zip`。所有 fixture 均为脱敏虚构数据。
