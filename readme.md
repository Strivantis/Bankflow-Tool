# 交易金流桑基圖工具 —— 入門到上手

這份說明會一步一步帶你從零開始：安裝 Miniconda、Visual Studio Code（藍色圖示）、建立 Python 環境與 Jupyter Kernel，最後教你如何修改參數並產生圖表。
不需要先會寫程式。照做即可。

---

## 0) 這個工具能做什麼

* 讀取 Excel 報表（含 `.xlsm` 巨集檔）。
* 自動清理欄位、辨識出入帳與對手方。
* 以 **桑基圖** 視覺化帳戶間金流，支援過濾金額、只取前 N 大交易、合併互相對沖等。
* 你只需要改一小段「可調參數」與最後的「繪圖選擇」。
* 也很歡迎以此為基底修改任何你想分析的東西喔！

---

## 1) 準備檔案結構

把專案資料夾放成這樣：

```
bankflow/
├─ 交易明細.xlsm              ← 你的原始交易明細
├─ tx_categories.yaml        ← 自訂分類規則（可自行設定分類規則，語法不會就去問 GPT）
├─ bankflow.ipynb            ← 我幫你準備的 Notebook
└─ README.md                 ← 本文件
```

> 你也可以轉換成 `bankflow.py` 來使用，但初學者推薦用 `bankflow.ipynb`。

---

## 2) 安裝 Miniconda（管理 Python 與套件）

1. 進入 Miniconda 官網下載安裝程式（Windows/macOS 都有）。
2. 安裝時一路預設即可。若有「加入 PATH」選項，勾選也可以，沒勾也不影響。

> 目的：用 **conda** 管理彼此隔離的 Python 環境，避免套件衝突。

---

## 3) 安裝 Visual Studio Code（藍色）

1. 下載並安裝 VS Code。
2. 開啟 VS Code 後，安裝以下兩個擴充套件：

   * `Python`（Microsoft 出品）
   * `Jupyter`（Microsoft 出品）

> 目的：在 VS Code 內即可跑 `.py` 和 `.ipynb`，按下「執行」就能跑。

---

## 4) 建立專用的 Conda 環境

在 **Visual Studio Code** 開啟終端機（快捷鍵 **Ctrl + Shift + `**），輸入：

```bash
# 建立 Python 環境，Python 版本 3.11（穩定常用）
conda create -n <Name> python=3.11 -y

# 啟用環境，下次你使用的時候，打開終端機後輸入這行即可
conda activate <Name>
```

> 看到前面有 `(<Name>)` 才是成功啟用。


---

## 5) 安裝必要套件

在 **已啟用** 的環境中執行：

```bash
pip install -U pip
pip install pandas plotly openpyxl pyyaml nbformat
```

> 說明：
>
> * `pandas` 資料處理
> * `plotly` 畫互動式桑基圖
> * `openpyxl` 讀 Excel
> * `pyyaml` 讀分類規則
> * `nbformat` 讓 Notebook 更順

---

## 6) 在 VS Code 選對解譯器與 Kernel

1. 用 VS Code 開啟專案資料夾 `bankflow/`。
2. 開啟 `bankflow.ipynb` 時，右上角會顯示 **Kernel**。點一下，選擇 `<Name>` 環境對應的 Kernel。

   * 第一次可能會問你「要不要為此環境建立 Jupyter Kernel」，按建立即可。

> 成功後，Notebook 每個區塊左邊會有一個「執行」按鈕。逐格執行就能看到結果。

---

## 7) 首次驗證

在 Notebook 新增一格並執行：

```python
import pandas as pd, plotly, yaml, openpyxl
print("OK")
```

沒有報錯且印出 `OK` 就通過。

---

## 8) 你的 Excel 與欄位對應

將原始交易明細放到 `bankflow.xlsx`。程式預設讀 `SHEET="Raw"`。
如果你的報表欄名不同，可在 `COLUMN_MAP` 調整同義欄位，例如：

```python
COLUMN_MAP = {
    "withdraw": ["支出金額", "轉出金額", "出金"],
    "deposit":  ["存入金額", "轉入金額", "入金"],
    "account":  ["帳號", "本行帳號"],
    "opponent": ["轉出入行庫代碼及帳號", "對方帳號", "交易對象"],
    "opponent_atm": ["ATM或末端機代號", "ATM代號"],
    "summary":  ["交易摘要", "摘要"],
    "remark":   ["備註", "附註", "說明"],
}
```

> 原理：程式會在清單中從左到右找第一個存在的欄位名稱。

---

## 9) 自訂分類規則（`tx_categories.yaml`）

當對手帳號判讀不到時，會用「摘要＋備註」嘗試分類。
`tx_categories.yaml` 長相示意：

```yaml
categories:
  - name: salary
    label: 薪資
    priority: 10
    direction: D        # D=入金, W=出金, any=不限
    patterns:
      - "薪資"
      - "匯入薪"
    negatives:
      - "薪資扣回"

  - name: atm_withdraw
    label: ATM提款
    priority: 20
    direction: W
    patterns:
      - "自提卡跨行提款"
      - "ATM"
```

* `priority` 越小越先比對。
* 命中任一 `patterns` 且不含 `negatives` 才成立。
* 若想要所有未命中的都落到「其他」，把程式上方參數 `ALLOW_FALLBACK_OTHER=True`。

---

## 10) 常用 Conda 指令速查

```bash
conda activate <Name>       # 啟用環境
conda deactivate              # 關閉環境
conda env list                # 列出所有環境
conda remove -n <Name> --all -y   # 刪除整個環境
```

---

## 11) 推薦用法：Notebook（最簡單）

1. 打開 `bankflow.ipynb`。
2. 設定完各參數後點擊 code 右上角有個播放圖示「執行」。
4. 看到輸出 `[ok] wrote: xxx.html`，會自動開啟瀏覽器看到桑基圖。
5. 想換不同帳戶視角，就改最後一格的 `view_accounts(...)` 參數再執行一次。

---
## 13) 你只需要修改哪兩段

### A. 檔頭「可調參數」

```python
FILE  = "bankflow.xlsm"        # 檔名
SHEET = "Raw"                 # 工作表名
TOP_NODE_K = 10               # 上色的前 K 節點
TOP_EDGES  = 20               # 取「金額總和」前 N 筆（None=不限制）
MIN_AMOUNT = 10001            # 最小金額過濾（None=不過濾）

# 欄位對應（若你的欄名不同，改這裡的同義詞清單）
COLUMN_MAP = { ... }

# 備註抽對手方用的模式（維持預設即可，除非你有新樣式要加）
CP_PATTERNS = [ ... ]
```

### B. 檔尾「畫哪張圖」

兩種選擇，擇一或都用：

1. **全帳戶圖**（則一取消註解即可）

```python
plot_sankey(
    sub_edges=_edges,
    title=f"All Flows (A-mode, node_scope={CATEGORY_NODE_SCOPE})",
    out_html=None,
    top_nodes=top_nodes
)
```

2. **指定帳戶的視角**

```python
view_accounts(
    edges=edges,
    accounts=["20181000619174", "28881006563125", "0222979189099"],  # 想看哪幾個帳號
    hops=1,                           # 看幾層關聯
    direction="both",                 # 下面應該都不用改，有興趣請問 GPT。
    include_categories=True,          
    min_amount=MIN_AMOUNT,            
    top_edges=TOP_EDGES,              
    per_account=False,                
    net_pairs=False                   
)
```

---

## 14) 互動圖的小技巧

* 滑鼠移過連線會顯示「來源 → 目標  金額」兩端提示。
* 可以自由移動節點。
* 視窗變動會自動調整寬度。圖檔輸出為 `.html`，可直接分享。
* 運行後會產生 `sankey_decisions_audit.csv` ，可以看到程式的分類邏輯，做必要的修正。

---

## 15) 一鍵檢查清單

* [ ] 安裝 Miniconda
* [ ] 安裝 VS Code 並加裝 Python、Jupyter 擴充
* [ ] `conda create -n Py3.11 python=3.11`
* [ ] `conda activate Py3.11`
* [ ] `pip install pandas plotly openpyxl pyyaml nbformat`
* [ ] 放好 `bankflow.xlsx` 與 `tx_categories.yaml`
* [ ] 在 VS Code 選 `Py3.11` Kernel
* [ ] 修改程式檔開頭參數與最後的繪圖選擇
* [ ] 執行並檢視輸出的 `.html` 圖表


---

