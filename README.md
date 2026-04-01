# 房屋數據缺失值插補方法比較分析 (Housing Data Missing Value Imputation Comparison Analysis)

## 專案簡介

本專案旨在對多種缺失值插補技術進行全面的比較分析，包括傳統的統計方法（平均值、中位數、眾數）、基於距離的 K-Nearest Neighbors (KNN) 方法，以及基於模型的 XGBoost 迭代插補。我們使用一個真實的房屋數據集，並從三個關鍵維度評估這些方法：

1.  **直接評估 (Direct Assessment)**：衡量插補值與真實值之間的數值誤差 (RMSE, MAE)、相關性 (Pearson r)、偏差 (Bias) 和變異性保留 (Variance Ratio)。
2.  **間接評估 (Indirect Evaluation)**：評估插補後的數據對下游機器學習任務（房價等級分類）性能的影響。
3.  **計算效率 (Computational Efficiency)**：記錄每種方法的總執行時間 (TCT)。

透過此分析，我們期望為不同應用場景下的缺失值處理提供實用的建議，並展示不同插補策略在數據品質和模型性能上的權衡。

## 資料集

本專案使用加州房屋數據的變體，包含 20,640 筆房屋資料和 10 個特徵。主要文件如下：

*   `housing_incomplete.csv`：包含隨機生成缺失值的實驗數據集。
*   `housing_complete.csv`：原始的完整數據集，作為真實值 (Ground Truth) 用於評估。
*   `missing_mask.json`：記錄了 `housing_incomplete.csv` 中所有缺失值的原始位置，用於精確評估。

### 關鍵變數定義：

*   **目標變數 (`TARGET_COL`)**：`median_house_value` (房屋中位數價值)。
*   **類別變數 (`CAT_COL`)**：`ocean_proximity` (與海洋的距離)。
*   **插補數值欄位 (`IMPUTE_COLS`)**：
    *   `longitude` (經度)
    *   `latitude` (緯度)
    *   `housing_median_age` (房屋中位數屋齡)
    *   `total_rooms` (房屋總房間數)
    *   `total_bedrooms` (房屋總臥室數)
    *   `population` (街區人口數)
    *   `households` (家庭總數)
    *   `median_income` (家庭中位數收入)

## 插補方法

### 1. 統計插補 (Statistical Methods)

*   **方法**：Mean (平均值), Median (中位數), Mode (眾數)
*   **原理**：用欄位本身的單一統計量填補缺失值。
*   **優點**：速度極快，實現簡單。
*   **缺點**：忽略特徵間關聯性，可能扭曲分佈，降低變異性。

### 2. KNN 插補 (K-Nearest Neighbors Imputation)

*   **方法**：K-Nearest Neighbors Imputer
*   **原理**：根據 K 個最相似鄰居的值來估計缺失值。
*   **實驗參數**：測試 `n_neighbors` = 3, 5, 10。
*   **最終選用**：`n_neighbors` = 5。
*   **優點**：考慮局部模式和特徵相似性。
*   **缺點**：計算成本較高，對數值尺度敏感（需特徵縮放）。

### 3. XGBoost 迭代插補 (XGBoost Iterative Imputation)

*   **方法**：XGBoost Regressor (迭代式填補)
*   **原理**：將每個含有缺失值的欄位視為迴歸問題，透過迭代訓練 XGBoost 模型來預測缺失值。
*   **實驗參數**：測試 `n_iter` = 3, 5, 10。
*   **最終選用**：`n_iter` = 5。
*   **優點**：捕捉複雜非線性關聯，插補精度極高，保留原始數據分佈特性。
*   **缺點**：計算成本最高，執行時間最長，需參數調優。

## 評估指標

### 1. 直接評估 (Direct Assessment)

衡量插補值與真實值在缺失點上的數值差異。

*   **RMSE (Root Mean Squared Error)**：均方根誤差，越低越好。
*   **MAE (Mean Absolute Error)**：平均絕對誤差，越低越好。
*   **Pearson r (Pearson Correlation Coefficient)**：皮爾遜相關係數，越接近 1 越好。
*   **Bias (偏差)**：插補值平均與真實值平均之差，越接近 0 越好。
*   **Variance Ratio (變異比)**：插補值變異數與真實值變異數之比，越接近 1.0 越好。

### 2. 間接評估 (Indirect Evaluation)

評估插補後的數據在下游機器學習任務（房價三分類：Low, Medium, High）上的表現。

*   **Accuracy (準確度)**：分類正確率，越高越好。
*   **F1-Score (Macro Average)**：平衡精確度和召回率，越高越好。
*   **Precision (精確度)**：越高越好。
*   **Recall (召回率)**：越高越好。

### 3. 效率評估 (Efficiency Assessment)

*   **TCT (Total Computation Time)**：總計算時間，越低越好。

## 主要結果與發現

| Method  | RMSE      | MAE       | Pearson r | Bias      | Var Ratio | Accuracy  | F1 (macro)| TCT (s)   |
| :------ | :-------- | :-------- | :-------- | :-------- | :-------- | :-------- | :-------- | :-------- |
| Mean    | 931.7568  | 324.4125  | 0.6977    | -6.6577   | 0.4651    | 0.6216    | 0.5999    | 0.0109    |
| Median  | 958.7619  | 307.7955  | 0.6976    | -126.5645 | 0.3062    | 0.6170    | 0.6053    | 0.0166    |
| Mode    | 1029.4563 | 328.8994  | 0.6891    | -221.1144 | 0.1759    | 0.6664    | 0.6631    | 0.0118    |
| KNN     | 733.8424  | 207.2341  | 0.8363    | -64.4810  | 0.5071    | 0.6923    | 0.6903    | 34.8461   |
| XGBoost | 263.9532  | 71.0095   | 0.9794    | -3.5079   | 0.9241    | 0.7970    | 0.7968    | 22.9486   |

**核心結論：**

*   **精度表現**：**XGBoost 迭代插補**在所有精度相關指標 (RMSE, MAE, Pearson r, Bias, Var Ratio, Accuracy, F1-Score) 上均表現最佳，插補結果最接近真實值，且對下游任務的性能提升最顯著。其 Bias 最小，Variance Ratio 最接近 1.0，表明其能良好地保留數據分佈。
*   **效率表現**：統計插補方法 (Mean, Median, Mode) 的執行時間極短，**Mean** 方法為最快。然而，它們在精度方面表現最差，且 Variance Ratio 遠低於 1.0，嚴重低估了數據的原始變異性。
*   **平衡性**：**KNN 插補**在精度上優於統計方法，但遜於 XGBoost。其執行時間介於統計方法和 XGBoost 之間。它在精度與計算成本之間提供了一個較好的平衡點。
*   **下游任務影響**：XGBoost 插補後的數據在房價分類任務中取得了最高的 Accuracy (0.7970) 和 F1-Score (0.7968)，這證明其不僅數值準確，也最大程度地保留了數據的結構完整性，有利於後續模型訓練。

## 視覺化圖表

專案中生成了多個圖表來輔助分析：

*   `knn_results.png`：KNN 插補不同 k 值下的 RMSE 和 R² 比較。
*   `statistical_results.png`：統計插補方法在各欄位上的表現，以及 `median_income` 分佈對比。
*   `xgboost_results.png`：XGBoost 迭代插補不同迭代次數下的表現、收斂曲線和特徵重要性熱圖。
*   `final_comparison.png`：所有方法在各項主要指標上的統一比較圖表，包含雷達圖。
*   `comprehensive_comparison.png`：更詳細的多維度綜合比較圖表，包含 RMSE、Bias、Variance Ratio、Pearson r、下游分類指標和計算時間。

## 如何運行 (How to Run)

1.  **克隆專案 (Clone the repository)**：
    ```bash
    git clone https://github.com/YourGitHubUsername/YourProjectName.git
    cd YourProjectName
    ```

2.  **安裝依賴 (Install dependencies)**：
    ```bash
    pip install pandas numpy scikit-learn matplotlib seaborn xgboost
    ```

3.  **準備數據 (Prepare Data)**：
    確保 `housing_incomplete.csv`, `housing_complete.csv`, `missing_mask.json` 三個文件位於專案根目錄下。這些文件通常會包含在專案資料夾中，如果沒有，您可能需要從原始數據來源下載。

4.  **執行 Jupyter Notebook / Colab**：
    開啟 `Missing_Value_Imputation_Comparison.ipynb` (或其他您主要執行的 .ipynb 文件) 並依序運行所有儲存格即可重現實驗結果和生成所有圖表及輸出文件。

## 輸出文件

專案執行後會生成以下 CSV 文件和 PNG 圖表：

*   `housing_knn_imputed_k5.csv`：KNN (k=5) 插補後的數據。
*   `housing_stat_mean_imputed.csv`：Mean 插補後的數據。
*   `housing_stat_median_imputed.csv`：Median 插補後的數據。
*   `housing_stat_mode_imputed.csv`：Mode 插補後的數據。
*   `housing_xgb_imputed_iter5.csv`：XGBoost (n_iter=5) 插補後的數據。
*   `imputation_final_summary.csv`：所有方法的主要評估指標摘要表。
*   `imputation_comprehensive_summary.csv`：更詳細的綜合評估指標摘要表。
*   `knn_results.png`
*   `statistical_results.png`
*   `xgboost_results.png`
*   `final_comparison.png`
*   `comprehensive_comparison.png`

## 結論與建議

根據多維度評估，本專案得出以下建議：

*   **精度優先**：若要求最高的插補精度和最佳的下游模型性能，**XGBoost 迭代插補**是最佳選擇。
*   **效率優先**：若對計算時間有嚴格要求（例如需要極快速的即時處理），統計方法中的 **Mean** 插補法最快，但需接受較低的精度和數據分佈失真。
*   **精度與速度平衡**：**KNN 插補**提供了一個良好的平衡點，在精度上顯著優於統計方法，且計算成本低於 XGBoost。

最終的插補策略應根據具體的業務需求、數據特性和計算資源限制來選擇。

---
