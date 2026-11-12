# 該怎麼設：照你的情況

預設是**沙盒開著、一般模式**：改檔先秀 diff 問你、白名單的指令直接跑、其他的問。大部分的人什麼都不用改。
兩個設定各管什麼、每個模式在沙盒開著和關掉時怎麼做，見 [usage.md](usage.md) 的「沙盒和模式」。

下面每一種寫出設定檔（`~/.xing-janus/config.json`）要加的那幾行，和狀態列會看到什麼。設定檔只寫要改的欄位，其他照預設；改了之後重開 xj。
常用的幾項（開 xj 時開沙盒、預設模式、允許切到全自動…）也可以在 xj 裡用 `/config` 改，或直接問 xj 該怎麼設（見 usage.md 的「改設定」）。

| 你的情況 | 沙盒 | 模式 |
|---|---|---|
| 平常開發 | 開 | 一般（或自動改檔） |
| 程式碼信得過、想少被問 | 開 | 全自動 |
| 讓它自己跑一段、不停下來等你 | 開 | 全部不問 |
| 要用 docker、flutter | 開：docker 列進 `excludedCommands`；flutter 內建就會在沙盒外跑（每次問） | 照上面選 |
| 可以整個丟掉的容器、虛擬機 | 關（沙盒起不來的話，那台的設定檔寫 `sandbox.enabled: false`） | 全部不問 |

## 平常開發

什麼都不用改。狀態列：「一般模式 · 沙盒開」。

- 第一次在某個資料夾開會問要不要信任：自己的專案選「是」（白名單的指令、事後檢查、AGENTS.md 才會用），別人的 repo 先選「否」（只能用唯讀、計劃模式；之後用 `/trust` 改）。
- 常跑的指令（`make lint` 這類）在詢問時按 `p`（永久允許）寫進白名單，之後直接跑；用 `/permissions` 看、拿掉。
- 改檔不想每次問：切到自動改檔（Shift-Tab；或設定檔 `"defaultMode": "auto-edit"`）。改了什麼照樣印出來，`/undo` 還原得了。

## 要用 Docker 這類沙盒裡跑不了的

不用整個關掉沙盒：把那幾個指令列進 `excludedCommands`，只有它們在沙盒外跑、每次都問。

```json
{ "sandbox": { "excludedCommands": ["docker"] } }
```

- flutter、dart、fvm、安裝程式（winget、scoop、brew）內建就會這樣（要在沙盒外跑時每次問），不用列。
- 細節見 [config.md](config.md) 的「Docker：在沙盒外跑的指令」「沙盒裡跑不了的工具（Flutter）」。

## 要讀、要改別的資料夾

在 A 開 xj、跟 A 有關的 B 也要讀或改：在 xj 裡打 `/add-dir ../B`（問只能讀還是也能改；也能改的要信任過）。
每次在 A 開都要一起加的，寫進設定檔：

```json
{ "permissions": { "addDirs": { "~/work/A": [{ "path": "~/work/B", "write": true }, { "path": "~/work/docs" }] } } }
```

這樣在 A 開 xj 時就加了 B（也能改）和 docs（只能讀），狀態列多一欄「資料夾 3」（連工作目錄一共幾個）。只有這一次要加的，啟動時加 `xj --add-dir ../B`（也能改）。
見 [usage.md](usage.md) 的「加進來的資料夾」、[config.md](config.md) 的「開 xj 時一起加的資料夾」。不要在上層（放所有專案的資料夾）開 xj：那樣每個專案都在範圍裡。

## 想少被問

全自動：改檔和沙盒裡的指令都不問，由沙盒當安全邊界（沙盒開著才不問指令）。設定檔先允許，之後用 Shift-Tab 切到它：

```json
{ "allowFullAuto": true }
```

- 只開這一次的話，啟動時加 `xj --auto`。只能在信任的資料夾用。
- 照樣問：危險的指令、新的網域、在沙盒外跑的指令、MCP 的工具、白名單以外的 PowerShell 指令（Windows）。
- **開始前先 git commit**：沙盒裡的指令照樣能改工作目錄，指令改的 `/undo` 還原不了。
- 狀態列：「全自動模式 · 沙盒開」。

## 讓它自己跑、不停下來問

全部不問、沙盒開著：什麼都不問，但指令照樣關在沙盒裡，沒允許過的網域不問就擋。

```json
{ "allowBypass": true, "sandbox": { "allowedDomains": ["registry.npmjs.org"] } }
```

- 之後 Shift-Tab 切得到，每次開 xj 第一次進去要打「我了解風險」；只開這一次的話，啟動時加 `xj --dangerously-skip-permissions`。
- 要連的網域先列進 `allowedDomains`：全部不問的時候，沒允許過的不問就擋。
- `excludedCommands` 列的（docker 這類）不問、直接在沙盒外跑；內建的 flutter、dart、fvm、安裝程式不問就擋：
  flutter 這類要用的話列進 `excludedCommands`，安裝程式換到會問你的模式、或你自己在終端機裡裝。
- 照樣問：刪掉重要的地方（根目錄、家目錄、工作目錄和它的上層…）、切換成 root、你自己加進 `permissions.danger` 的。
- **開始前先 git commit。** 預算的上限照樣有效（`/budget`）。
- 狀態列：「全部不問模式 · 沙盒開」。

## 在可以整個丟掉的容器、虛擬機裡

全部不問、沒有沙盒：什麼都不問，指令以你的權限跑。**只在可以整個丟掉的環境裡用。**

```json
{ "allowBypass": true, "defaultMode": "bypass", "sandbox": { "enabled": false } }
```

- `defaultMode: "bypass"` 是開 xj 就進全部不問（照樣先打「我了解風險」，取消的話從一般模式開始）；不寫這一項的話用 Shift-Tab 切過去。
- 容器裡沙盒常常起不來（起不來的話一律拒絕跑指令）：那台的設定檔寫 `sandbox.enabled: false`，或啟動時加 `--no-sandbox`。
- 用 root 跑時全部不問開不了；一定要的話設環境變數 `XING_JANUS_ALLOW_ROOT_BYPASS=1`。
- 機密檔、受保護的檔案只擋得到指令裡直接寫出名字的（`grep -r` 這類間接的讀法擋不到）：機密別放在這個資料夾。
- 狀態列整列標紅（兩個都放到最寬）。

## Windows 還沒裝沙盒

照樣能用：每個指令都先問你，你答應了才以你的權限在沙盒外跑（白名單不生效）。狀態列：「沙盒沒裝好」。
要裝的話見 [install.md](install.md) 的「Windows 的安裝」；`xj doctor` 講裝好了沒、還缺什麼。

## Linux 的沙盒起不來

沙盒起不來時一律拒絕跑指令（狀態列：「沙盒起不來」）。先跑 `xj doctor` 看原因，常見的見 [troubleshooting.md](troubleshooting.md)。
暫時要用的話：在 xj 裡打 `/sandbox off`，或用 `xj --no-sandbox` 開（每個指令都問）。
