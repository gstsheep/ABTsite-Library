# 流程圖總覽

所有流程圖以 Mermaid 撰寫，原始碼位於 `flowcharts/*.mmd`，並輸出 SVG / PNG。GitHub 可直接渲染下方 Mermaid 區塊。

> 標示【推定】者為依 CFC 腳位名稱推論之邏輯，實際行為以 Library 模塊為準。

## 目錄

1. [新案場建置流程](#1-新案場建置流程)
2. [ABTsite 資料階層](#2-abtsite-資料階層)
3. [冰水系統 CFC 資料流](#3-冰水系統-cfc-資料流)
4. [冰水系統加減機邏輯【推定】](#4-冰水系統加減機邏輯推定)
5. [冰水系統階段啟動 / 停機程序【推定】](#5-冰水系統階段啟動--停機程序推定)
6. [保護機制與跳脫機制](#6-保護機制與跳脫機制)
7. [冷卻系統 CFC 資料流](#7-冷卻系統-cfc-資料流)
8. [冷卻系統頻率與加減機邏輯【推定】](#8-冷卻系統頻率與加減機邏輯推定)
9. [故障排除決策流程](#9-故障排除決策流程)

## 1. 新案場建置流程

圖檔：[SVG](../flowcharts/01-deployment-workflow.svg) ／ [PNG](../flowcharts/01-deployment-workflow.png)

```mermaid
flowchart TD
    A(["開始：新案場建置"]) --> B["取得完整程式檔案<br/>ABT site Library.s1cx / Sample.ABTArchive"]
    B --> C["備份原始資料<br/>確認輸出資料夾寫入權限"]
    C --> D["ABTsite → Library → Import<br/>匯入 ABT site Library.s1cx"]
    D --> E{"Library 是否出現<br/>Blocks / Modbus / Plants / Points<br/>四個子資料夾？"}
    E -- 否 --> E1["重新匯入 / 確認 ABTsite 版本 ≥ v5.2"] --> D
    E -- 是 --> F{"首次使用？"}
    F -- 是 --> F1["Restore project → Sample<br/>對照範例資料練習"] --> G
    F -- 否 --> G["Building structure<br/>建立 Building / Floor / Area / Subarea / Section"]
    G --> H["Show automation stations<br/>依 IP 設定表建立控制器<br/>Name / IP / Mask / Gateway / Instance"]
    H --> I["Show rooms & plants<br/>建立 Plant：CWS / CHWS / CTS / *S<br/>指定 Device（控制器）"]
    I --> J["準備 IO 表：列出需求點位"]
    J --> K{"需求點位是否存在於<br/>IO Tag.xlsx / Library Points？"}
    K -- 否 --> K1["協同維護者於 IO Tag.xlsx<br/>新增並定義點位"] --> K
    K -- 是 --> L["Engineering → Plants<br/>拖曳 Points / Modbus 點位"]
    L --> M{"系統是否符合模板？<br/>冰機 ≤ 6、水塔 ≤ 4<br/>非 DXR / 非空調箱"}
    M -- 是 --> N["套用 Plants 模板<br/>冰水系統 / 冷卻系統"]
    M -- 否 --> M1["自行撰寫 CFC<br/>或以 Blocks 自行拼組"] --> O
    N --> O["設定參數<br/>加減機溫度、間隔、確認時間、除冷時間等"]
    O --> P["Check → Download 至控制器"]
    P --> Q["驗證 A：Engineering → connect → play<br/>檢查數值與程式運行"]
    Q --> R["驗證 B：Desigo CC 指定畫面<br/>確認 reference 對應與系統流程"]
    R --> S{"運轉結果正常？"}
    S -- 否 --> T["依故障排除表處理"] --> O
    S -- 是 --> U(["完成：交付 / 歸檔"])
```

## 2. ABTsite 資料階層

圖檔：[SVG](../flowcharts/02-data-hierarchy.svg) ／ [PNG](../flowcharts/02-data-hierarchy.png)

```mermaid
flowchart LR
    P["Project<br/>ABTsite 專案"] --> B["Building<br/>自訂義或案場代碼<br/>例：N（XXX科技廠 N場）"]
    B --> F1["Floor<br/>樓層代碼<br/>B1 / 01F / 02F / 03F"]
    F1 --> AR["Area<br/>Ar + 流水號"]
    AR --> SA["Subarea<br/>sbAr + 流水號"]
    SA --> SC["Section<br/>sc + 流水號"]
    F1 --> AS["Automation stations 視圖<br/>key：控制器位置<br/>例：N-B1F-001 PXC7.E400M"]
    F1 --> RP["Rooms & plants 視圖<br/>key：供應或應用的區域<br/>例：CHWS 冰水系統"]
    RP -. "Device 指派" .-> AS
    RP ==> CC["Desigo CC 依此路徑建立"]
```

## 3. 冰水系統 CFC 資料流

圖檔：[SVG](../flowcharts/03-chws-dataflow.svg) ／ [PNG](../flowcharts/03-chws-dataflow.png)

```mermaid
flowchart LR
    subgraph IN["輸入點位"]
        CMD["總系統起停點<br/>CMD_B"]
        NUM["可運轉系統數量<br/>W_DINT"]
        RT["冰水總回水溫度計<br/>R_A"]
        TA["加機溫度 W_A"]
        TS["減機溫度 W_A"]
        IA["加機間隔 W_DINT"]
        IS["減機間隔 W_DINT"]
        CHD["CH1~CH6<br/>En / 運轉時數 / 效率"]
    end
    SEQ["冰機順位比較模塊<br/>CFC"]
    ADD["Add-ons and Subtractions<br/>加減機模塊 AddnSub<br/>CFC"]
    CHD --> SEQ
    SEQ -- "順位1~6冰機 → Sq1~Sq6" --> ADD
    CMD -- StartStop --> ADD
    NUM -- Num --> ADD
    RT -- T --> ADD
    TA -- "T Add-on" --> ADD
    TS -- "T Sub-dn" --> ADD
    IA -- "Add time" --> ADD
    IS -- "Sub time" --> ADD
    ADD -- "SYS1~SYS6" --> SYSn
    subgraph SYSn["每一組系統 n（n = 1…6）"]
        PRT["保護機制 Prt.Cnf_n<br/>CFC"]
        OL["跳脫機制 OL.Cnf_n<br/>Overload configuration"]
        RS["系統階段啟動 Sys.rs_n<br/>System run step"]
        PRT -- "status → Protection CFC" --> RS
        OL -- "OL → Alarm CFC" --> RS
    end
    RS --> OUT["冰水泵系統 / 區域泵系統 /<br/>冷卻水泵系統 / 水塔系統 / 冰機起停"]
    CTS["冷卻系統 CTS<br/>水塔系統狀態 / CT 跳脫狀態"] -. "水塔狀態 / 水塔OL" .-> PRT
    CTS -. "水塔OL" .-> OL
```

## 4. 冰水系統加減機邏輯【推定】

圖檔：[SVG](../flowcharts/04-chws-staging.svg) ／ [PNG](../flowcharts/04-chws-staging.png)

```mermaid
flowchart TD
    S(["每個掃描週期"]) --> A{"總系統起停點<br/>StartStop = ON？"}
    A -- 否 --> Z["SYS1~SYS6 全部 OFF<br/>各系統進入停機程序"]
    A -- 是 --> B["冰機順位比較模塊<br/>依 En、運轉時數、效率<br/>排出 Sq1~Sq6"]
    B --> C["讀取冰水總回水溫度 T"]
    C --> D{"T ≥ 加機溫度<br/>且 運轉台數 < 可運轉系統數量 Num？"}
    D -- 是 --> E{"持續時間 ≥ 加機間隔<br/>Add time？"}
    E -- 是 --> F["依順位啟動下一組系統<br/>SYS(n+1) = ON"]
    E -- 否 --> W["維持現況"]
    D -- 否 --> G{"T ≤ 減機溫度<br/>且 運轉台數 > 1？"}
    G -- 是 --> H{"持續時間 ≥ 減機間隔<br/>Sub time？"}
    H -- 是 --> I["依順位關閉最後啟動之系統<br/>SYS(n) = OFF"]
    H -- 否 --> W
    G -- 否 --> W
    F --> S
    I --> S
    W --> S
    Z --> S
```

## 5. 冰水系統階段啟動 / 停機程序【推定】

圖檔：[SVG](../flowcharts/05-chws-run-step.svg) ／ [PNG](../flowcharts/05-chws-run-step.png)

```mermaid
flowchart TD
    subgraph START["啟動程序（SYSn = ON）"]
        direction TB
        S0(["system ON/OFF = ON"]) --> S1{"Protection CFC status 正常<br/>且 Alarm CFC 無 OL？"}
        S1 -- 否 --> SX["禁止啟動 / 發出警報"]
        S1 -- 是 --> S2["① 冰水泵系統 ON"]
        S2 --> S3["② 區域泵系統 ON"]
        S3 --> S4["③ 冷卻水泵系統 ON"]
        S4 --> S5["④ 水塔系統 ON"]
        S5 --> S6{"開機確認時間內<br/>泵狀態 + 流量開關 確認？"}
        S6 -- 否 --> SX
        S6 -- 是 --> S7["⑤ 冰機起停 ON"]
    end
    subgraph STOP["停機程序（SYSn = OFF 或保護動作）"]
        direction TB
        T0(["system ON/OFF = OFF"]) --> T1["① 冰機起停 OFF"]
        T1 --> T2["② 等待 關機除冷間隔<br/>（SYSn 除冷時間）"]
        T2 --> T3["③ 水塔 / 冷卻水泵 / 區域泵 / 冰水泵 OFF"]
    end
    SX --> T1
```

## 6. 保護機制與跳脫機制

圖檔：[SVG](../flowcharts/06-protection-trip.svg) ／ [PNG](../flowcharts/06-protection-trip.png)

```mermaid
flowchart TD
    subgraph P["保護機制 Prt.Cnf_n（CFC）"]
        direction LR
        P1["系統起停<br/>← AddnSub.SYSn"]
        P2["開機確認時間<br/>← SYSn 確認時間"]
        P3["維修點<br/>← SYSn 維修點"]
        P4["冰水泵狀態 / 冰水流量開關<br/>← CHPn 狀態 / 流量開關"]
        P5["冷卻水泵狀態 / 冷卻水流量開關<br/>← CWPn 狀態 / 流量開關"]
        P6["水塔狀態<br/>← 水塔狀態.PrVal"]
    end
    subgraph O["跳脫機制 OL.Cnf_n（Overload configuration）"]
        direction LR
        O1["冰水泵OL<br/>← CHPn 跳脫"]
        O2["冷卻水泵OL<br/>← CWPn 跳脫"]
        O3["水塔OL<br/>← 水塔OL.PrVal"]
        O4["冰機OL<br/>← CHn 跳脫"]
    end
    P -- "status → Protection CFC" --> RS["系統階段啟動 Sys.rs_n"]
    O -- "OL → Alarm CFC" --> RS
    RS --> R1{"維修中 / 狀態不符 / 任一 OL？"}
    R1 -- 是 --> R2["該組系統停機或禁止啟動<br/>由加減機模塊依順位補位"]
    R1 -- 否 --> R3["允許正常運轉"]
```

## 7. 冷卻系統 CFC 資料流

圖檔：[SVG](../flowcharts/07-cts-dataflow.svg) ／ [PNG](../flowcharts/07-cts-dataflow.png)

```mermaid
flowchart LR
    CMD["水塔系統起停點<br/>CMD_B"]
    LT["冷卻水總出水溫度計<br/>R_A (TM)"]
    subgraph G1["頻率計算輸入"]
        direction TB
        DB["外氣乾球溫度計 R_A"]
        RH["外氣相對濕度計 R_A"]
        AP["趨近溫度設定 W_A"]
        HI["輸出高限 W_A"]
        LO["輸出低限 W_A"]
        DIR["CTR 正反向 W_B"]
    end
    subgraph G2["加減機輸入"]
        direction TB
        TA["加機溫度 W_A"]
        TS["減機溫度 W_A"]
        NUM["可運轉系統數量 W_DINT"]
        IA["加機間隔 W_DINT"]
        IS["減機間隔 W_DINT"]
        ST["CT1~CT4 變頻驅動狀態 R_B"]
    end
    TR["CT1~CT4 變頻驅動跳脫 R_B"]
    FQ["水塔頻率計算<br/>CFC"]
    AD["水塔加減機<br/>CFC"]
    TJ["水塔跳脫判別<br/>CFC"]
    CMD -- 水塔系統起停 --> FQ
    CMD -- "cmd_b_1 → 水塔系統起停" --> AD
    LT -- 出水溫 --> FQ
    LT -- "TM → 出水溫" --> AD
    G1 --> FQ
    G2 --> AD
    TR -- "CT1~CT4 OL" --> TJ
    FQ -- "頻率 → ValIn1 Prio 16" --> VFD["CT1~CT4 變頻驅動控制<br/>CMD_A"]
    AD -- "CTn 起停 → ValIn1 Prio 16" --> SS["CT1~CT4 變頻驅動起停<br/>CMD_B"]
    AD -- CT狀態 --> CS["水塔系統狀態<br/>W_B"]
    TJ -- CT跳脫狀態 --> CO["CT s 跳脫狀態<br/>W_B"]
    CS -. "Interconnection" .-> EXT["冰水系統 CHWS<br/>保護機制 / 跳脫機制<br/>CTs.con.出水溫 / 水塔系統"]
    CO -. "Interconnection" .-> EXT
```

## 8. 冷卻系統頻率與加減機邏輯【推定】

圖檔：[SVG](../flowcharts/08-cts-control.svg) ／ [PNG](../flowcharts/08-cts-control.png)

```mermaid
flowchart TD
    S(["每個掃描週期開始"]) --> A{"水塔系統起停 = ON？"}
    A -- 否 --> Z["CT1~CT4 起停 OFF<br/>頻率輸出 = 0"] --> END
    A -- 是 --> F1["【頻率計算】以乾球溫度 + 相對濕度<br/>求外氣濕球溫度 Twb"]
    F1 --> F2["目標出水溫 = Twb + 趨近溫度SET"]
    F2 --> F3["比較 冷卻水總出水溫 與 目標值<br/>依 CTR 正反向 決定調節方向"]
    F3 --> F4["頻率輸出限制於 輸出低限 ~ 輸出高限"]
    F4 --> A1{"【加減機】出水溫 ≥ 加機溫度<br/>持續 ≥ 加機間隔<br/>且運轉台數 < 使用台數？"}
    A1 -- 是 --> A2["啟動下一台可用 CT<br/>（略過跳脫 / 狀態異常者）"]
    A1 -- 否 --> A3{"出水溫 ≤ 減機溫度<br/>持續 ≥ 減機間隔？"}
    A3 -- 是 --> A4["關閉一台 CT"]
    A3 -- 否 --> A5["維持台數"]
    A2 --> J
    A4 --> J
    A5 --> J
    J["【跳脫判別】彙整 CT1~CT4 OL<br/>→ CT s 跳脫狀態"]
    J --> O["輸出：CTn 變頻驅動起停 / 控制（Prio 16）<br/>水塔系統狀態、CT 跳脫狀態 → 冰水系統"]
    O --> END(["等待下一掃描週期"])
```

## 9. 故障排除決策流程

圖檔：[SVG](../flowcharts/09-troubleshooting.svg) ／ [PNG](../flowcharts/09-troubleshooting.png)

```mermaid
flowchart TD
    A(["發現問題"]) --> B{"需求 IO 不在 Library？"}
    B -- 是 --> B1["查雲端 IO Tag.xlsx 是否已定義"] --> B2{"已定義？"}
    B2 -- 是 --> B3["由 Library Points 對應資料夾拖曳<br/>（部分資料需於 Programming 的 Library 拉入）"]
    B2 -- 否 --> B4["協同維護者新增並定義點位<br/>更新 IO Tag.xlsx 與 Library"]
    B -- 否 --> C{"實際程式需求與模板不符？"}
    C -- 是 --> C1{"冰機 > 6 / 水塔 > 4 /<br/>DXR / 空調箱 / 能源運算？"}
    C1 -- 是 --> C2["超出適用範圍：自行撰寫 CFC"]
    C1 -- 否 --> C3["以 Blocks 自行拼組模塊"]
    C -- 否 --> D{"CC 畫面無數值？"}
    D -- 是 --> D1["確認 Rooms & plants 路徑<br/>與 CC reference 對應"]
    D -- 否 --> E{"Engineering 無法 connect？"}
    E -- 是 --> E1["核對控制器 IP / Mask / Gateway / Instance<br/>與 IP 設定表一致"]
    E -- 否 --> F["Email 回報維護者<br/>Phill.lin@siemens.com"]
```
