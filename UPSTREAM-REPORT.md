# 小红书图集图片 404：可复现案例及修复

## 现象

在 res-downloader 4.0.0-beta.7 中抓取并下载小红书图集时，同一组里部分图片在 `sns-webpic-qc.xhscdn.com` 返回 HTTP 404；将相同签名路径及查询参数用于 `sns-webpic-backup.xhscdn.com` 后，可下载同一张图片。其他图片在默认地址正常。实际签名链接会过期，并可能含敏感参数，因此不附真实链接或抓包。

## 复现

1. 在 res 中打开抓取，并打开小红书图文作品。
2. 下载一个至少有一张 QC CDN 图片地址返回 404 的图集。
3. 未修复的下载器会在默认地址失败后报告该图片下载失败，而其他图片可能成功。

## 修复

插件保留原始 URL，并为图片下载计划标记 `xiaohongshuBackup`。配套主程序先请求原始地址，只在 404、服务端错误或网络错误时重试备用 CDN；403、用户取消及本地文件错误不触发切换。两个来源使用不同的临时文件和断点，避免混用。插件执行是同步的，单独修改插件无法观察宿主的下载失败。

- 插件补丁：<https://github.com/2008Terry/resd-plugin-xiaohongshu-fallback/tree/fix/conditional-image-fallback>
- 主程序补丁（针对 `release/v4.0`）：<https://github.com/2008Terry/res-downloader-xhs-fallback/tree/fix/xhs-image-cdn-fallback>

## 验证

- 修复版 4.0.0-beta.7 实际下载了一篇 15 张图片的图集及其文案，16 个任务全部成功，15 张图片均有 RIFF/WEBP 文件头。
- 一张图片的默认地址成功；另一张默认地址返回 404，随后从备用地址成功下载。
- `go test ./internal/download -run '^TestXHS' -count=1` 在上游 `release/v4.0` 提交 `2d600c5` 的补丁副本上通过。
- 具有独立 ID 的派生插件通过契约测试与插件规范检查。

本方案不保证过期链接、权限拒绝或所有未来 CDN 变化均可下载。官方采纳、发布和合并由维护者决定。

