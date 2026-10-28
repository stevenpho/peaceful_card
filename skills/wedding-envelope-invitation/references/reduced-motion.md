# 減少動態效果與驗證

## 實作原則

- 禁止以全域倍率（例如 `K = reduce ? 0.01 : 1`）壓縮開卡計時；這會讓 CSS 淡出尚未完成，JS 就移除遮罩。
- 改用固定時間分支，讓 reduced-motion 保留短暫停留與淡入淡出。此專案目前為點擊後 700ms 開始淡出，淡出 650ms 後顯示內容／解鎖捲動，開始淡出 1000ms 後移除遮罩；一般模式保留原時序。數值隨 CSS 調整，清理不可早於淡出完成。
- CSS 的 reduce 覆寫應放在相關一般規則之後：停用呼吸、循環旋轉、推進、翻蓋及縮放；明確指定 `opacity`、`transform`、`scale`、`translate`、`filter` 等最終狀態。不可只移除 animation，卻留下初始隱藏值。
- `.fx`／`.art.fx` 等捲動淡入元素在 reduce 下設為 `opacity:1; transform:none; scale:1; translate:none; filter:none; transition:none`，確保圖片與底部內容可見。
- 純裝飾粒子在建立前跳過；花瓣啟動函式在 reduce 時直接 return。使用者主動選擇的圖集內容與裝飾特效分開處理，不因停用裝飾而失去內容。
- JS 的偏好目前於載入時讀取；CSS 媒體查詢會隨設定更新。測試中途切換系統設定後，重新載入頁面，避免誤認為兩者即時同步。

## 驗證

1. 一般模式及 reduce 模式都檢查開卡前、淡出中、完成後、兩個朋友選項、圖集與圖片。確認遮罩移除、捲動解鎖，且未重複進場。
2. 桌面替代測試：僅在暫時副本將相關 reduce 媒體查詢改為 `@media all`，並強制 JS reduce 分支；僅改 CSS 無法驗證 JS 計時。保持素材路徑有效，測完刪除副本，不提交測試覆寫。
3. iOS 模擬器：先記錄原設定與選定裝置 ID，再用 `xcrun simctl spawn <ID> defaults write com.apple.Accessibility ReduceMotionEnabled -bool true`；在 Safari 小測試頁確認 `matchMedia('(prefers-reduced-motion: reduce)').matches` 實際為 true，不能只以命令成功推定。測完還原原設定；原值為 false 才寫回 false。若系統未更新偏好，應標記該次驗證未成立。
4. 實體 iPhone：與電腦同一 Wi-Fi，透過電腦區網 IP 開啟預覽；`127.0.0.1` 在手機指向手機本身。需要區網測試才將預覽伺服器綁定至區網介面或 `0.0.0.0`。記錄 Safari／App 內建瀏覽器、機型與 iOS 版本。
5. 回報分清桌面強制分支、模擬器 Safari、實體機測試及尚未測試項目；commit、push、部署狀態分開陳述，不自動推送或發佈。

## 修正後仍回報異常

先核對 commit 是否已 push、部署是否包含修正及實際載入版本，再排查 Safari 快取。沒有直接證據時，只能將舊快取列為可能原因。確認是否由 LINE／FB／IG 內建瀏覽器開啟，請以 Safari 比較；仍異常再取得機型、iOS 版本與錄影並重現。
