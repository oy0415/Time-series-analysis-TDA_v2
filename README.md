# Time-series-analysis-TDA_v2

## 要約

UCI Individual Household Electric Power Consumption データを用いて、家庭の電力消費量を対象とした時系列分析を行いました。  
本プロジェクトでは、前処理・EDA、統計的時系列モデル、機械学習モデル、Darts による LSTM、さらに Topological Data Analysis（TDA）による特徴量作成と評価までを一連の流れとして実装しています。

分析の結果、今回の条件では **通常特徴量のみを用いた GradientBoostingRegressor** が最も高い予測性能を示しました。  
一方で、TDA特徴量を追加した場合、今回の設定では明確な精度改善は確認されませんでした。  
ただし、一部のTDA特徴量は特徴量重要度を持っており、時系列の形状情報を一定程度表現している可能性があります。

---

## プロジェクト概要

このプロジェクトでは、UCI Individual Household Electric Power Consumption データを用いて、家庭の電力消費量 `Global_active_power` の時系列予測を行いました。

最初に、1分単位の元データを読み込み、欠損値、データ型、時系列インデックス、重複時刻、時刻間隔などを確認しました。  
その後、日次データへ集約し、統計的時系列モデル、機械学習モデル、深層学習モデルを段階的に比較しました。

さらに、Sliding Window / Takens Embedding と Persistent Homology を用いて、時系列データからTDA特徴量を作成しました。  
作成したTDA特徴量を既存の時系列特徴量に追加し、予測性能が改善するかを検証しました。

本分析では、単に高精度なモデルを作るだけでなく、以下の点を重視しています。

- 時系列データとしての前処理を丁寧に行うこと
- 統計モデル、機械学習モデル、深層学習モデルを段階的に比較すること
- TDA特徴量が時系列予測に有効かを検証すること
- データリークを避けた特徴量設計を行うこと
- 精度が改善しなかった場合も、その結果を考察として整理すること

---

## 使用データ

- **データ名**: UCI Individual Household Electric Power Consumption
- **使用ファイル**: `data/raw/household_power_consumption.txt`
- **データ単位**: 1分間隔の家庭電力消費データ
- **分析単位**: 日次データ
- **主な目的変数**: `Global_active_power`

### 主な変数

| 変数名 | 内容 |
|---|---|
| `Global_active_power` | 家庭全体の有効電力 |
| `Global_reactive_power` | 家庭全体の無効電力 |
| `Voltage` | 電圧 |
| `Global_intensity` | 電流 |
| `Sub_metering_1` | サブメータリング1 |
| `Sub_metering_2` | サブメータリング2 |
| `Sub_metering_3` | サブメータリング3 |

---

## 分析の流れ

本プロジェクトでは、以下の流れで分析を行いました。

1. データの読み込み
2. 欠損値・データ型・時系列インデックスの確認
3. 1分単位データから日次データへの集約
4. 可視化による時系列構造の確認
5. 統計的時系列モデルによる予測
6. ラグ特徴量・移動統計量・カレンダー特徴量の作成
7. 機械学習モデルによる予測
8. Darts を用いた LSTM モデルによる予測
9. Sliding Window / Takens Embedding による時系列の点群化
10. Persistent Homology によるTDA特徴量の作成
11. TDA特徴量を追加したモデルとの比較
12. 結果の整理と考察

---

## Notebook構成

| Notebook | 内容 |
|---|---|
| `01_preprocessing_eda.ipynb` | データの読み込み、欠損値確認、日次集約、EDA |
| `02_statistical_time_series.ipynb` | 統計的時系列モデルとベースラインモデルの比較 |
| `03_machine_learning_forecasting.ipynb` | ラグ特徴量・移動統計量・カレンダー特徴量を用いた機械学習モデルの比較 |
| `04_deep_learning_forecasting.ipynb` | Darts を用いた LSTM による深層学習モデルの実装と評価 |
| `05_tda_feature_engineering_and_evaluation.ipynb` | TDA特徴量の作成、可視化、通常特徴量との比較評価 |

---

# 1. 前処理・EDA

## 目的

`01_preprocessing_eda.ipynb` では、元データを読み込み、時系列データとして分析できる形に整えることを目的としました。

主に以下を確認しました。

- データ型
- 欠損値
- 重複時刻
- 時刻間隔
- 各変数の基本統計量
- `Global_active_power` の推移
- 日次データへの集約

---

## 主な処理

元データでは、`Date` と `Time` が別々の列として与えられているため、これらを結合して `datetime` 型の時系列インデックスを作成しました。

また、欠損値を確認したうえで、後続の分析に使いやすいように日次平均データを作成しました。

作成したデータは、後続のnotebookで利用できるように保存しました。

---

## 結果

前処理により、1分単位の元データから日次データを作成しました。  
これにより、統計的時系列モデル、機械学習モデル、深層学習モデル、TDA特徴量作成に共通して利用できるデータを準備できました。

---

# 2. 統計的時系列モデル

## 目的

`02_statistical_time_series.ipynb` では、統計的時系列モデルを用いて、機械学習モデルや深層学習モデルと比較するための基準を作成しました。

比較対象として、平均値によるベースラインモデルや指数平滑化系のモデルを扱いました。

---

## 評価指標

主に以下の指標を用いて予測性能を評価しました。

- MAE
- MSE
- RMSE
- MASE

---

## 結果

ベースラインモデルの比較では、平均値を用いたモデルが以下の性能を示しました。

| モデル | MAE | RMSE |
|---|---:|---:|
| mean baseline | 0.257821 | 0.330350 |

一方で、指数平滑化系のモデルでは、今回の条件ではベースラインよりも良い結果にはなりませんでした。

---

## 解釈

統計的時系列モデルは、時系列予測の基準として有用でした。  
ただし、今回のデータでは、単純な統計モデルだけでは十分に複雑な変動を捉えきれない可能性があります。

そのため、次の分析では、ラグ特徴量や移動統計量を作成し、機械学習モデルによる予測へ進みました。

---

# 3. 機械学習モデル

## 目的

`03_machine_learning_forecasting.ipynb` では、日次データに対してラグ特徴量、移動統計量、カレンダー特徴量を作成し、複数の機械学習モデルを比較しました。

---

## 作成した主な特徴量

- ラグ特徴量
- 移動平均
- 移動標準偏差
- 曜日
- 月
- トレンド特徴量

---

## 比較したモデル

| モデル | 内容 |
|---|---|
| LinearRegression | 線形回帰モデル |
| RandomForestRegressor | ランダムフォレスト |
| GradientBoostingRegressor | 勾配ブースティング |
| LightGBM | 勾配ブースティング木モデル |
| detrended model | トレンド除去後に予測し、予測後にトレンドを戻すモデル |

---

## 結果

機械学習モデルでは、通常特徴量を用いた `GradientBoostingRegressor` が最も良い性能を示しました。

| モデル | MAE | RMSE | MAPE |
|---|---:|---:|---:|
| GradientBoostingRegressor | 0.167023 | 0.238333 | 0.184404 |
| LightGBM_detrended | 0.177449 | 0.248498 | 0.200940 |
| LightGBM | 0.177627 | 0.249516 | 0.198671 |
| RandomForestRegressor | 0.180173 | 0.245607 | 0.204136 |
| LinearRegression | 0.182355 | 0.246311 | 0.211172 |
| LinearRegression_detrended | 0.185141 | 0.247649 | 0.219039 |

---

## 解釈

機械学習モデルでは、ラグ特徴量や移動統計量を使うことで、統計的時系列モデルよりも予測性能が改善しました。

特に、`GradientBoostingRegressor` は非線形な関係を捉えやすく、今回のデータでは最も良い結果となりました。  
一方で、トレンド除去を行ったモデルが必ずしも性能改善につながるわけではなく、今回の条件では通常特徴量のみのモデルが有力でした。

---

# 4. 深層学習モデル

## 目的

`04_deep_learning_forecasting.ipynb` では、時系列分析ライブラリ Darts を用いて、LSTM による深層学習モデルを実装しました。

通常の LSTM、ハイパーパラメータ調整後の LSTM、future covariates を用いた LSTM を比較しました。

---

## 比較したモデル

| モデル | 内容 |
|---|---|
| LSTM | 基本的な LSTM モデル |
| LSTM tuning | ハイパーパラメータ調整後の LSTM |
| LSTM + future covariates | カレンダー特徴量などの将来既知の特徴量を用いた LSTM |

---

## 結果

Darts による LSTM では、future covariates を加えたモデルが最も良い性能を示しました。

| モデル | データ | MAE | RMSE | MAPE |
|---|---|---:|---:|---:|
| LSTM | validation | 0.554695 | 0.640241 | 40.911639 |
| LSTM tuning | validation | 0.232300 | 0.304500 | 20.590500 |
| LSTM tuning | test | 0.394803 | 0.482642 | 59.730131 |
| LSTM + future covariates | test | 0.195529 | 0.253708 | 25.363071 |

---

## 解釈

ハイパーパラメータ調整や future covariates の追加により、LSTM の性能は改善しました。  
特に、曜日や月などの将来既知の特徴量を用いることで、時系列の周期性を補足できた可能性があります。

ただし、今回の条件では、機械学習モデルの `GradientBoostingRegressor` よりも高い性能にはなりませんでした。  
この結果から、深層学習モデルは有効な選択肢ではあるものの、データ量、特徴量設計、モデル設定によっては木ベースモデルの方が高い性能を示す場合があると考えられます。

---

# 5. TDA特徴量作成と評価

## 目的

`05_tda_feature_engineering_and_evaluation.ipynb` では、時系列データに対して Topological Data Analysis（TDA）を適用し、作成したTDA特徴量が予測性能の改善につながるかを検証しました。

具体的には、Sliding Window / Takens Embedding によって時系列を点群化し、Persistent Homology によって Persistence Diagram や Barcode を作成しました。  
その後、Persistence Diagram から数値特徴量を抽出し、機械学習モデルに追加しました。

---

## TDA特徴量作成の流れ

1. 時系列データから rolling window を作成
2. 各 window に対して Takens Embedding を適用
3. 点群データを作成
4. Persistent Homology を計算
5. Persistence Diagram / Barcode を確認
6. Persistence Diagram から数値特徴量を作成
7. 通常特徴量にTDA特徴量を追加
8. 通常特徴量のみのモデルと比較

---

## データリークへの対応

TDA特徴量は、rolling window の終了時点までの情報を使って作成されます。  
そのため、同じ日の予測にそのまま使うと、将来情報を含む可能性があります。

本分析では、`tda_available_date = rolling_window_end_date + 1日` とし、TDA特徴量が翌日以降の予測にのみ使われるようにしました。  
これにより、データリークを避けた形でTDA特徴量を評価しました。

---

## 結果

通常特徴量のみのモデルと、TDA特徴量を追加したモデルを比較した結果、今回の条件ではTDA特徴量の追加による明確な精度改善は確認されませんでした。

| モデル | 通常特徴量 RMSE | TDA追加 RMSE | RMSE差 |
|---|---:|---:|---:|
| GradientBoosting | 0.220544 | 0.220767 | +0.000223 |
| RandomForest | 0.222327 | 0.222641 | +0.000314 |
| Ridge | 0.228476 | 0.237015 | +0.008539 |

最良モデルは、通常特徴量のみを用いた `GradientBoostingRegressor` でした。

| モデル | 特徴量 | MAE | RMSE | MAPE | MASE | R² |
|---|---|---:|---:|---:|---:|---:|
| GradientBoosting | normal only | 0.158254 | 0.220544 | 17.633341 | 0.603824 | 0.446169 |

---

## 解釈

今回の設定では、TDA特徴量を追加しても予測精度は改善しませんでした。  
そのため、少なくとも今回の window 幅、embedding dimension、delay、特徴量化方法では、通常特徴量を上回る効果は限定的だったと考えられます。

一方で、一部のTDA特徴量は特徴量重要度を持っていたため、時系列の形状情報を一定程度表現している可能性があります。  
今後は、TDA特徴量の設計やパラメータを変更することで、より有効な特徴量を作成できる可能性があります。

---

# モデル比較のまとめ

各notebookの主な結果をまとめると、以下のようになります。

| Notebook | 主なモデル | MAE | RMSE | コメント |
|---|---|---:|---:|---|
| `02_statistical_time_series.ipynb` | mean baseline | 0.257821 | 0.330350 | 統計モデル比較の基準 |
| `03_machine_learning_forecasting.ipynb` | GradientBoostingRegressor | 0.167023 | 0.238333 | 機械学習モデルで大きく改善 |
| `04_deep_learning_forecasting.ipynb` | LSTM + future covariates | 0.195529 | 0.253708 | LSTMでは最良だが、機械学習モデルには届かず |
| `05_tda_feature_engineering_and_evaluation.ipynb` | GradientBoosting normal only | 0.158254 | 0.220544 | 全体で最良 |

今回の分析では、最終的に **通常特徴量のみを用いた GradientBoostingRegressor** が最も高い予測性能を示しました。

---

## 結論

本プロジェクトでは、UCI Individual Household Electric Power Consumption データを用いて、家庭電力消費量の時系列予測を行いました。

前処理・EDAから始め、統計的時系列モデル、機械学習モデル、深層学習モデル、TDA特徴量を用いたモデルまでを段階的に比較しました。  
その結果、今回の条件では、通常特徴量のみを用いた `GradientBoostingRegressor` が最も良い性能を示しました。

また、TDA特徴量を追加した場合、今回の設定では明確な予測精度の改善は確認されませんでした。  
しかし、TDA特徴量は時系列の形状的な情報を表現する可能性があり、パラメータや特徴量化方法を変更することで、異なる結果が得られる可能性があります。

---

## 考察

今回の分析では、時系列データに対して段階的にモデルを発展させることで、各手法の特徴と限界を確認することができました。

統計的時系列モデルは、予測性能の基準として有用でしたが、今回のデータの変動を十分に捉えるには限界がありました。  
一方で、ラグ特徴量や移動統計量を用いた機械学習モデルでは、予測性能が大きく改善しました。  
特に、GradientBoostingRegressor は、電力消費量の非線形な変動を比較的よく捉えたと考えられます。

深層学習モデルでは、future covariates を加えることで性能が改善しました。  
ただし、今回の条件では機械学習モデルを上回る結果にはなりませんでした。  
このことから、深層学習モデルを有効に使うには、データ量、特徴量設計、モデル構造、学習条件をさらに検討する必要があると考えられます。

TDA特徴量については、今回の設定では予測精度の改善にはつながりませんでした。  
しかし、TDAは時系列の形状情報を扱える手法であり、window幅、embedding dimension、delay、Persistence Diagram からの特徴量抽出方法を変更することで、より有効な特徴量を作成できる可能性があります。

今後は、TDA特徴量の設計をさらに検討し、どのような時系列構造に対してTDAが有効に働くのかを追加で検証したいです。

---

## 今後の課題

今後は、以下のような改善が考えられます。

- TDA特徴量の window 幅を変更する
- Takens Embedding の embedding dimension や delay を調整する
- Persistence Diagram から抽出する特徴量を増やす
- Persistence Image や Betti Curve など、別のTDA特徴量を検討する
- LightGBM や GradientBoosting のハイパーパラメータをさらに調整する
- 深層学習モデルで GRU や N-BEATS などのモデルも比較する
- 予測対象を日次平均以外に変更して検証する
- 外れ値や電力消費ピーク日の分析を追加する

---

## ファイル構成

```text
Time-series-analysis-TDA/
├── data/
│   ├── raw/
│   │   └── household_power_consumption.txt
│   └── processed/
│       ├── household_power_consumption_cleaned.csv
│       ├── household_power_consumption_daily.csv
│       ├── Global_active_power_daily.csv
│       └── tda_features.csv
│
├── notebooks/
│   ├── 01_preprocessing_eda.ipynb
│   ├── 02_statistical_time_series.ipynb
│   ├── 03_machine_learning_forecasting.ipynb
│   ├── 04_deep_learning_forecasting.ipynb
│   └── 05_tda_feature_engineering_and_evaluation.ipynb
│
├── src/             # 今後追加予定
├── README.md
└── requirements.txt
```
※元データは容量の都合によりGitHubには含めていません。取得方法は data/data_README.md を参照してください。

---

## 使用ライブラリ

```text
pandas
numpy
matplotlib
seaborn
scikit-learn
statsmodels
lightgbm
darts
torch
giotto-tda
```

---

## 補足

このプロジェクトでは、単に予測精度を比較するだけでなく、以下の点を重視しました。

- 時系列データとして適切に前処理すること
- 統計的時系列モデルをベースラインとして扱うこと
- 機械学習・深層学習・TDA特徴量を段階的に比較すること
- データリークを避けて特徴量を設計すること
- TDA特徴量で精度が改善しなかった結果も、検証結果として正直に整理すること

そのため、本プロジェクトは、時系列予測の基本的な流れに加えて、TDA特徴量の有効性検証までを一貫して示す内容になっています。
