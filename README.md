# Linux Kernel Patch 提交指南 (Traditional Chinese)

這是一個以繁體中文撰寫的 Linux Kernel Patch 提交與貢獻指南。本專案以官方文件 [`Documentation/process/submitting-patches.rst`](https://www.kernel.org/doc/html/latest/process/submitting-patches.html) 為基礎，整理了實用的工作流程、Git 操作規範以及社群互動守則，幫助開發者更順利地將程式碼貢獻至開源社群。

## 📖 指南內容涵蓋

- **原始碼與開發樹**：如何透過 `MAINTAINERS` 尋找子系統特定的開發樹（例如 DRM 框架與驅動）。
- **Patch 撰寫與拆分**：Commit message 規範（祈使句、`Fixes` tag 等）、程式碼風格檢查 (`checkpatch.pl`)。
- **寄送 Patch 與信件禮儀**：`git send-email` 設定與純文字格式回覆規範 (Interleaved reply)。
- **審查標籤 (Review Tags)**：`Signed-off-by`、`Acked-by`、`Reviewed-by` 等各類標籤的意義與使用時機。
- **現代化工具 (`b4`)**：專門為 Kernel 開發設計的工具指南（涵蓋 `prep`、`send`、`trailers` 指令）。

## 🚀 如何使用

專案內包含：
- **`linux_patch_guide.md`**：Markdown 格式的完整指南，適合直接透過編輯器或 GitHub 瀏覽。
- **`linux_patch_guide.html`**：HTML 格式的網頁版本文件，提供不同的離線閱讀體驗。

## 👨‍💻 作者

- **Hermes.wu@ite.com.tw** & **Antigravity**

---
*If you find this guide helpful, feel free to contribute or suggest improvements!*
