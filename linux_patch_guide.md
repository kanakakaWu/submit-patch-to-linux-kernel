# Linux Kernel Patch 提交指南

> **作者：** Hermes.wu@ite.com.tw & Antigravity

---

## 1. 取得最新原始碼

> **說明：** 儘管實際開發使用的版本不一定是最新的，但是 patch 提交建議仍以 kernel 最新版本或是 maintainer tree 的最新版本為基礎。

從主線倉庫 clone：

```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

> [!NOTE]
> 大多數子系統 maintainer 有自己的樹（tree），請優先針對該樹開發。查詢方式：`MAINTAINERS`（或 `MAINTAINERS.txt`）檔案中的 **T:** 欄位。

### 範例：以 DRM 子系統查詢開發樹

**步驟 1 — 搜尋 MAINTAINERS**

假設你要修改 `drivers/gpu/drm/` 下的程式碼：

```bash
# 方法 A：直接搜尋關鍵字
grep -n "^DRM" MAINTAINERS | head -20

# 方法 B：讓工具自動找（推薦）
scripts/get_maintainer.pl --scm your.patch
```

**步驟 2 — 閱讀 MAINTAINERS 區塊**

在 `MAINTAINERS` 中找到對應區塊（以 DRM 主框架為例，實際位於 `MAINTAINERS.txt` 第 8449 行附近）：

```
DRM DRIVERS
M:	David Airlie <airlied@gmail.com>
M:	Simona Vetter <simona@ffwll.ch>
L:	dri-devel@lists.freedesktop.org
S:	Maintained
T:	git https://gitlab.freedesktop.org/drm/kernel.git    ← 主 DRM 樹
F:	drivers/gpu/drm/
```

DRM misc 驅動（panel、tiny driver 等）則使用子樹：

```
DRM PANEL DRIVERS
M:	Neil Armstrong <neil.armstrong@linaro.org>
L:	dri-devel@lists.freedesktop.org
S:	Maintained
T:	git https://gitlab.freedesktop.org/drm/misc/kernel.git   ← misc 樹
F:	drivers/gpu/drm/panel/
```

各欄位含義：

| 欄位 | 說明 |
|------|------|
| `M:` | Maintainer（主要聯絡人） |
| `L:` | 郵件列表（送 patch 時需 CC） |
| `S:` | 狀態（Maintained / Supported / Orphan…） |
| `T:` | **開發樹 git URL**（針對此樹開發） |
| `F:` | 負責的檔案/目錄 |

**步驟 3 — Clone 並建立開發分支**

```bash
# 以 DRM misc 樹為例（panel / 小型驅動修改）
git clone https://gitlab.freedesktop.org/drm/misc/kernel.git drm-misc
cd drm-misc

# 追蹤遠端 drm-misc-next 分支
git checkout -t origin/drm-misc-next
```

**步驟 4 — 確認送件目標**

```bash
# 產生 patch 後確認收件人
git format-patch -1
scripts/get_maintainer.pl 0001-drm-panel-fix-foo.patch
# 輸出範例：
# Neil Armstrong <neil.armstrong@linaro.org> (maintainer)
# dri-devel@lists.freedesktop.org (open list)
```

> [!TIP]
> DRM 子系統有兩棵常用樹：\
> • **`drm/kernel.git`** → DRM 核心框架\
> • **`drm/misc/kernel.git`** → panel、bridge driver、tiny driver、misc 驅動\
> 修改時需確認你的檔案對應哪棵樹的 **F:** 欄位。

---

## 2. 描述你的變更

良好的 commit 描述應包含：

| 要素 | 說明 |
|------|------|
| **問題說明** | 描述根本原因，說服 reviewer 這問題值得修復 |
| **使用者可見影響** | crash、鎖死、效能下降、延遲等具體現象 |
| **量化數據** | 效能/記憶體改善需附數字佐證 |
| **技術細節** | 用英文說明你實際做了什麼 |

**撰寫規則：**

- 使用 **祈使語氣**：`"make xyzzy do frotz"`，而非 `"I changed xyzzy"`
- **一個 patch 只解決一個問題**；描述太長代表該拆分
- 每次 resubmit 都要附上**完整說明**，不可只說「這是第 N 版」
- 引用 commit 時，至少用 **12 字元的 SHA-1** 加上單行摘要：
  ```
  Commit e21d2170f366 ("video: remove unnecessary platform_set_drvdata()")
  ```

**常用 Tag（放在描述結尾）：**

```
Link:   https://lore.kernel.org/<message-id>   # 相關討論
Closes: https://example.com/issues/1234        # 關閉的 bug 報告
Fixes:  54a4f0239f2e ("KVM: MMU: make kvm_mmu_zap_page() return ...")
```

> [!TIP]
> 設定 git 以快速產生 `Fixes:` 標籤：
>
>     [core]
>         abbrev = 12
>     [pretty]
>         fixes = Fixes: %h ("%s")
>
> 使用：`git log -1 --pretty=fixes <commit-id>`

---

## 3. 拆分變更

- **一個邏輯變更 = 一個 patch**
- Bug fix 與效能改善應分開送
- API 新增與使用該 API 的驅動程式應分開送
- 確保每個 patch 套用後 kernel 仍可正常建置與執行（避免 `git bisect` 困境）
- 單次投稿上限約 **15 個 patch**，等待 review 後再送下一批

---

## 4. 風格檢查

```bash
scripts/checkpatch.pl <your.patch>
```

報告等級：

| 等級 | 意義 |
|------|------|
| `ERROR` | 幾乎確定有問題 |
| `WARNING` | 需仔細審查 |
| `CHECK` | 需思考是否合適 |

> [!IMPORTANT]
> 移動程式碼時，**不要在同一個 patch 修改**被移動的程式碼，這樣才能清楚區分「移動」與「修改」兩個動作。

---

## 5. 選擇收件人

```bash
scripts/get_maintainer.pl <your.patch>
```

**收件原則：**

| 情境 | 收件人 |
|------|--------|
| 一般 patch | `linux-kernel@vger.kernel.org` + 子系統 maintainer |
| 安全漏洞 | `security@kernel.org`（勿公開發送） |
| stable 修復 | 在 sign-off 區加 `Cc: stable@vger.kernel.org` |
| API 影響 userspace | `linux-api@vger.kernel.org` + MAN-PAGES maintainer |
| 找不到 maintainer | `akpm@linux-foundation.org`（最後手段） |

---

## 6. 郵件格式：純文字、內嵌送出

> [!WARNING]
> **不要** 用 MIME 附件、壓縮檔或連結方式送 patch。

- 使用 **`git send-email`**（強烈建議）
  - 互動式教學：https://git-send-email.io
- Patch 必須以**純文字內嵌**方式傳送
- 郵件主旨加上 `[PATCH]` 前綴（`git send-email` 自動處理）

主旨格式：
```
Subject: [PATCH 001/123] subsystem: summary phrase
Subject: [PATCH v2 01/27] x86: fix eflags tracking
Subject: [PATCH Vx RESEND] sub/sys: Condensed patch summary   # 未修改重送
```

---

## 7. 回應 Review

- **一定要回覆** reviewer 的意見，忽略 reviewer 等於被忽略
- 對不導致程式碼變更的問題，也要回覆或更新 changelog
- 感謝 reviewer 的時間
- 使用**交錯回覆（interleaved reply）**，不要頂貼（top-posting）
- 修改版本時在 cover letter 或個別 patch 加上 **patch changelog**
- 把 reviewer 加到新版本的 CC 清單

**等待時間：**
- 一般等 **2-3 週**，忙碌期（merge window）更長
- 最少等 **一週** 再 resend 或 ping

---

## 8. Patch 標準格式

### 8.1 完整結構

```
Subject: [PATCH N/M] subsystem: summary phrase (≤75 字元)

From: Patch Author <author@example.com>    ← 僅在與寄件人不同時需要

<說明主體，75 欄換行，這部分會進入 git log>

Signed-off-by: Your Name <your@email.com>
---
<版本變更說明，不進入 git log>
<diffstat>

<diff 內容>
```

### 8.2 Backtrace

在 commit message 中放 backtrace 時，應**精簡**只保留關鍵呼叫鏈：

```
unchecked MSR access error: WRMSR to 0xd51
Call Trace:
mba_wrmsr
update_domains
rdtgroup_mkdir
```

### 8.3 版本資訊位置

版本 changelog 必須放在 `---` **之後**（不進入 git commit）：

```
Signed-off-by: Author <author@mail>
---
V2 -> V3: Removed redundant helper function
V1 -> V2: Cleaned up coding style

v2: https://lore.kernel.org/bar
v1: https://lore.kernel.org/foo

path/to/file | 5 +++--
```

---

## 9. Sign-off 與各種 Tag

### Developer's Certificate of Origin（DCO）

加上 `Signed-off-by:` 即表示你認證此貢獻符合 DCO 1.1 規定：

```
Signed-off-by: Your Name <your@email.com>
```

用 `git commit -s` 或 `git revert -s` 自動附加。

### Tag 一覽

| Tag | 說明 | 需明確授權？ |
|-----|------|------------|
| `Signed-off-by:` | 參與開發或傳遞路徑 | ✅ 是 |
| `Acked-by:` | 審閱並同意（可選加 `# 備注`） | ✅ 是 |
| `Reviewed-by:` | 正式技術審查通過 | ✅ 是 |
| `Tested-by:` | 測試環境驗證通過 | ✅ 是 |
| `Co-developed-by:` | 共同作者（需緊跟 SoB） | ✅ 是 |
| `Reported-by:` | 回報 bug 的人 | ❌ 可隱式授權 |
| `Suggested-by:` | 提出點子的人 | ❌ 可隱式授權 |
| `Cc:` | 知情方（未留言） | ❌ 可隱式授權 |
| `Fixes:` | 指向被修復的 commit | — |
| `Closes:` | 關閉的 bug 連結 | — |
| `Link:` | 相關討論連結 | — |

> [!CAUTION]
> 除 `Cc:`、`Reported-by:`、`Suggested-by:` 外，其他 tag 都需要**當事人明確同意**才能添加。

### Co-developed-by 範例

```
Co-developed-by: First Co-Author <first@coauthor.example.org>
Signed-off-by: First Co-Author <first@coauthor.example.org>
Co-developed-by: Second Co-Author <second@coauthor.example.org>
Signed-off-by: Second Co-Author <second@coauthor.example.org>
Signed-off-by: From Author <from@author.example.org>
```

### Reviewed-by 說明

`Reviewed-by` 標籤代表審查者認為該 patch 是對 kernel 適當的修改，且沒有殘留嚴重的技術問題。任何有進行技術審查的相關人員皆可提供此標籤。這不僅能給予審查者信用（credit），也讓 maintainer 了解該 patch 經過了何種程度的審查。來自熟悉該領域且審查嚴謹的 reviewer 所提供的 `Reviewed-by` 標籤，通常能大額提高 patch 被接受進入 kernel 的機率。

> [!NOTE]
> 當你在 mailing list 上收到 `Tested-by` 或 `Reviewed-by` 標籤時，發送**下一個版本**時應主動將這些標籤加入對應的 patch 中。
>
> **重要特例：**若該 patch 在後續版本有**大幅度變更**，原有的標籤可能已不適用，此時必須將其移除；且移除別人的 `Tested-by` 或 `Reviewed-by` 標籤時，務必在 patch 的 changelog（`---` 分隔線後）中說明原因。

---

## 10. Base Tree 資訊

讓 reviewer 與 CI 知道你的 patch 基於哪個 commit：

```bash
git checkout -t -b my-topical-branch master
# ... 進行修改 ...
git format-patch --base=auto --cover-letter -o outgoing/ master
```

產生的 patch 會自動在結尾附上：
```
base-commit: <commit-hash>
```

---

## 11. In-Reply-To

- 可用 `In-Reply-To:` 將 patch 與 bug 報告郵件關聯
- **多 patch series 不建議**用 In-Reply-To 指向舊版本（避免郵件樹混亂）
- 改用 cover letter 中放 lore.kernel.org 連結

---

## 12. 工具

> [!TIP]
> `b4` 工具已包含大部份工具的功能，強烈建議學習使用。如果新手容易搞混格式或流程問題，它能幫你自動打理好大部分細節！

| 工具 | 用途 |
|------|------|
| `git send-email` | 傳送 patch 郵件 |
| `git format-patch` | 產生標準格式 patch 檔 |
| `scripts/checkpatch.pl` | 風格與錯誤檢查 |
| `scripts/get_maintainer.pl` | 找出 patch 的正確收件人 |
| `b4` | 自動化流程（追蹤依賴、checkpatch、格式化、送信） → https://b4.docs.kernel.org |

---

## 13. b4 工具使用指南

`b4` 是 Linux kernel 社群維護的自動化工具，可簡化 patch 的準備、寄送與 trailer 回收流程。官方文件：<https://b4.docs.kernel.org>

### 13.1 安裝

```bash
# 推薦使用 pipx（獨立隔離環境）
pipx install b4

# 或透過套件管理器
dnf install b4      # Fedora/RHEL
apt install b4      # Debian/Ubuntu
```

---

### 13.2 開發者工作流程（b4 prep → send → trailers）

```
b4 prep -n <name>        建立 topical branch
  ↓ 撰寫 commits
b4 prep --edit-cover     編輯 cover letter
b4 prep --auto-to-cc     自動收集收件人
b4 prep --check          執行 pre-flight 檢查
b4 send -o /tmp/preview  預覽（不實際送出）
b4 send                  實際送出
  ↓ 等待 review
b4 trailers -u           回收 Reviewed-by / Acked-by
  ↓ git rebase -i 修改
b4 send                  送出下一版（自動遞增版號）
```

---

### 13.3 b4 prep — 準備 patch series

#### 建立新的 topical branch

```bash
# 從目前 HEAD 建立（自動建立 b4/<name> 分支並切換過去）
b4 prep -n fix-drm-panel-init

# 指定 fork point（建議從 tag 或已知 commit 分叉）
b4 prep -n fix-drm-panel-init -f drm-misc-next
```

> [!NOTE]
> `<name>` 會成為 change-id 的一部分，請取有意義的名稱，勿使用 `foo`、`test`。
> （註解：建立新分支時會出現一個新的 commit，主要記錄 b4 歷程，無程式碼變更）

#### 合入變更

準備好分支後，可透過 `git cherry-pick <commit>` 或 `git rebase -i` 將你的修改放入此分支中。

> [!TIP]
> \`git rebase -i\` 也可以用來在送出前修正 commit message。

> [!WARNING]
> \`cover letter\`（也就是記錄 b4 track 歷程的那個 commit）**必須**保持在這個 patch series 的第一個 commit（也就是最底層的 commit），不可將其往後移。

#### 編輯 cover letter

```bash
b4 prep --edit-cover      # 以 $EDITOR 開啟，存檔後自動更新
```

Cover letter 的第一行（Subject）會成為整個 series 的主旨；其餘說明寄送時放在
cover email 的 body，不會進入個別 commit。

> [!NOTE]
> 當 PATCH 只有一個時，不需要 cover letter，`b4 send` 送信時也不會發送 cover letter 作為 [PATCH 0/1]。

#### 收集收件人

```bash
b4 prep --auto-to-cc      # 自動查詢 MAINTAINERS，填入 To/Cc
```

產生後請人工檢視並移除不相關的地址。

> [!TIP]
> 可以使用 `git log <commit>` 來檢視編輯完成後的 cover letter 內容。

#### 其他常用選項

```bash
b4 prep --check           # 執行 checkpatch 等 pre-flight 檢查
b4 prep --cleanup         # 刪除已合入/過期的 prep 分支
```

---

### 13.4 b4 send — 送出 patch

#### 設定寄件方式

**方式 A：使用 SMTP（`.git/config` 或 `~/.gitconfig`）**

```
[sendemail]
    smtpServer     = smtp.example.org
    smtpServerPort = 465
    smtpEncryption = ssl
    smtpUser       = alice@example.org
    smtpPass       = <password>
```

**方式 B：使用 kernel.org Web 端點（適合未設定 SMTP 者）**

對於 kernel.org 託管的專案，可使用官方提供的 Web 端點寄信：

1.  **產生 ed25519 金鑰（若已有設定 `user.signingKey` 的 PGP 金鑰則可跳過）**：

        patatt genkey

    執行後將產生新金鑰，並在終端機輸出類似以下的設定提示：

        [patatt]
            signingkey = ed25519:20220915
            selector = 20220915

    請複製終端機上顯示的 `[patatt]` 區塊，並將其加入到你的全域 `~/.gitconfig` 或專案底下的 `.git/config` 中。

2.  **設定 Web 端點 URL**（在 `~/.gitconfig` 或 `.git/config`）：

        [b4]
            send-endpoint-web = https://lkml.kernel.org/_b4_submit

3.  **申請授權驗證**：

        b4 send --web-auth-new

    *這將產生一封包含驗證碼信件，發送到你的 `user.email`。*

4.  **完成驗證**：

    收取該郵件，執行郵件中提供的驗證指令：

        b4 send --web-auth-verify <challenge-string>
授權完成後，後續即可直接使用 `b4 send` 寄出 patch，無須透過自己的 SMTP 伺服器。

#### 送出流程

```bash
# 1. 預覽：輸出到目錄，不實際送信
b4 send -o /tmp/preview-patches

# 2. 寄一份給自己確認格式
b4 send --reflect

# 3. 正式送出
b4 send
```

送出後 b4 會自動：
- 將 changelog 追加到 cover letter
- 版號遞增（`v1` → `v2`）
- 在 lore.kernel.org 留下記錄

#### 重送（RESEND）

```bash
b4 send --resend          # 未修改內容，重送同版本
```

---

### 13.5 b4 trailers — 回收 Review 標籤

Reviewer 在郵件中回覆 `Reviewed-by:` / `Acked-by:` 後，用此指令自動抓回並
附加到對應的 commit：

```bash
b4 trailers -u            # 從 lore 抓取最新 trailers 並套用
```

- 比對依據：commit 的 **patch-id** 或 **標題** 相符即可自動配對
- 套用後若需調整，用 `git rebase -i` → `reword` 手動編輯

---

### 13.6 完整範例（DRM panel 修復）

```bash
# 1. 取得 DRM misc 開發樹
git clone https://gitlab.freedesktop.org/drm/misc/kernel.git drm-misc
cd drm-misc
git checkout -t origin/drm-misc-next

# 2. 建立 topical branch
b4 prep -n fix-drm-panel-foo -f drm-misc-next

# 3. 進行修改並 commit
vim drivers/gpu/drm/panel/panel-foo.c
git add -p
git commit -s       # -s 自動加 Signed-off-by

# 4. 編輯 cover letter
b4 prep --edit-cover

# 5. 收集收件人 & 預檢
b4 prep --auto-to-cc
b4 prep --check

# 6. 預覽後送出
b4 send -o /tmp/out   # 先看輸出
b4 send               # 確認無誤後正式送出

# --- 等待 review，收到 Reviewed-by 回覆後 ---

# 7. 回收 trailers
b4 trailers -u

# 8. 若需修改，rebase 後送下一版（版號自動遞增）
git rebase -i drm-misc-next
b4 send

# 9. 合入後清理
b4 prep --cleanup
```

---

## 參考資料

- [`Documentation/process/submitting-patches.rst`](https://www.kernel.org/doc/html/latest/process/submitting-patches.html)
- [The perfect patch – Andrew Morton](https://www.ozlabs.org/~akpm/stuff/tpp.txt)
- [Linux kernel patch submission format – Jeff Garzik](https://web.archive.org/web/20180829112450/http://linux.yyz.us/patch-format.html)
- [On submitting kernel patches – Andi Kleen](http://halobates.de/on-submitting-patches.pdf)
- [`Documentation/process/coding-style.rst`](https://www.kernel.org/doc/html/latest/process/coding-style.html)
- [`Documentation/process/email-clients.rst`](https://www.kernel.org/doc/html/latest/process/email-clients.html)
- [`Documentation/process/stable-kernel-rules.rst`](https://www.kernel.org/doc/html/latest/process/stable-kernel-rules.html)
- [`Documentation/process/security-bugs.rst`](https://www.kernel.org/doc/html/latest/process/security-bugs.html)
