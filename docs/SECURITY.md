# 回報安全問題

發現 Xing Janus 的安全問題（例如繞過 Cerberus、讀到機密檔、在沙盒外執行），請不要開公開的 issue。

請在 releases repo 的 **Security → Report a vulnerability** 私下回報（GitHub 的 private vulnerability reporting），寫清楚：

- 版本（`xj --version`）和 `xj doctor --report` 的輸出
- 怎麼重現（最好是一個小的範例專案）
- 你預期會發生什麼、實際發生了什麼

不要附上真的金鑰或機密的檔案內容。Xing Janus 擋得住什麼、擋不住什麼，見 [cerberus.md](cerberus.md)。
