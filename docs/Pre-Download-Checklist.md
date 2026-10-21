# 下載至控制器前檢查表

案場：__________　控制器：__________　檢查人：__________　日期：__________

## A. 專案與 Library

- [ ] ABTsite 版本 ≥ v5.2
- [ ] 已匯入 ABT site Library，且 Blocks / Modbus / Plants / Points 四個資料夾存在
- [ ] 已備份原始專案

## B. Building structure

- [ ] Building / Floor / Area / Subarea / Section 命名符合規則（Ar / sbAr / sc + 流水號）
- [ ] 控制器 Name、Description、Equipment ID 依 IP 設定表
- [ ] IP、Mask、Gateway、Device instance、Host name 與 IP 設定表一致
- [ ] Plant 名稱符合規則（CWS/CHWS、HWS、BS、CTS、`*S`），同系統不同控制器已加 extension
- [ ] Plant 已指派正確 Device

## C. 點位

- [ ] IO 表所有需求點位皆可在 IO Tag.xlsx / Library Points 找到
- [ ] 新增點位已與維護者確認並更新 IO Tag.xlsx
- [ ] 點位命名格式 `[類型] 中文功能` 正確
- [ ] 實體 IO / Modbus mapping 位址已核對

## D. 冰水系統（CHSYS）

- [ ] 冰機台數 ≤ 6，系統組數與 SYSn 接線一致
- [ ] 冰機順位比較模塊 En1~En6 依實際可用台數設定
- [ ] 可運轉系統數量、加 / 減機溫度、加 / 減機間隔已設定（非 0）
- [ ] 每組 SYSn 確認時間、除冷時間已設定
- [ ] 每組 CHPn / CWPn 狀態、流量開關、跳脫及 CHn 跳脫已接線
- [ ] 水塔狀態、水塔 OL 已接至每組保護 / 跳脫機制

## E. 冷卻系統（CTSYS）

- [ ] 水塔台數 ≤ 4
- [ ] 外氣乾球溫度、相對濕度、冷卻水總出水溫度計已對應實體點
- [ ] 趨近溫度、輸出高限 / 低限、CTR 正反向已設定
- [ ] 加 / 減機溫度、使用台數、加 / 減機間隔已設定
- [ ] CT1~CT4 起停、控制、狀態、跳脫已接線
- [ ] 跨控制器 reference（水塔狀態、CT 跳脫狀態）已建立

## F. 下載與驗證

- [ ] Check 無錯誤
- [ ] 確認下載不影響現場運轉（可重啟或終止）
- [ ] Engineering → connect → play，數值與程式運行正常
- [ ] Desigo CC 指定畫面 reference 已對應，數值與系統流程正常
- [ ] 未修改 CFC 內部邏輯
