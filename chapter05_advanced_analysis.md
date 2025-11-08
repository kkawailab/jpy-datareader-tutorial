# 第5章: 上級編 - データ分析の実践

## 📚 この章で学ぶこと

- 複数統計表の結合テクニック
- 時系列データの分析
- 地域比較分析
- データ可視化の実践
- 実践的なプロジェクト例

**対象レベル**: 上級者
**所要時間**: 約90分
**前提知識**: 第1〜4章の内容、pandas、matplotlib/seabornの基礎

---

## 5.1 複数統計表の結合

### 5.1.1 基本的な結合

```python
"""
複数の統計表を結合する
"""
import pandas as pd
from jpy_datareader.estat import StatsDataReader
from dotenv import load_dotenv

load_dotenv()

def merge_statistics(stats_id1, stats_id2, merge_keys):
    """
    2つの統計表を結合

    Parameters:
    -----------
    stats_id1 : str
        統計表ID 1
    stats_id2 : str
        統計表ID 2
    merge_keys : list
        結合キーのリスト

    Returns:
    --------
    pandas.DataFrame
        結合されたデータ
    """
    reader = StatsDataReader()

    # データ1を取得
    print(f"統計表1 [{stats_id1}] を取得中...")
    df1 = reader.read(statsDataId=stats_id1, limit=1000)
    print(f"✅ {len(df1)}件取得")

    # データ2を取得
    print(f"\n統計表2 [{stats_id2}] を取得中...")
    df2 = reader.read(statsDataId=stats_id2, limit=1000)
    print(f"✅ {len(df2)}件取得")

    # 共通カラムを確認
    common_cols = set(df1.columns) & set(df2.columns)
    print(f"\n共通カラム: {len(common_cols)}個")
    print(f"  {', '.join(list(common_cols)[:5])}...")

    # 結合
    print(f"\n結合キー: {merge_keys}")
    df_merged = pd.merge(
        df1, df2,
        on=merge_keys,
        how='inner',  # 内部結合
        suffixes=('_1', '_2')  # 同名カラムに接尾辞を付ける
    )

    print(f"✅ 結合完了: {len(df_merged)}件")

    return df_merged


# 使用例（実際の統計表IDに置き換えてください）
# stats_id1 = "0003410379"
# stats_id2 = "0003348423"
# merge_keys = ['地域コード', '時間コード']  # 実際のカラム名に合わせる
# df_merged = merge_statistics(stats_id1, stats_id2, merge_keys)
```

---

## 5.2 時系列分析

### 5.2.1 時系列データの準備と可視化

```python
"""
時系列データの分析
"""
import pandas as pd
import matplotlib.pyplot as plt
from jpy_datareader.estat import StatsDataReader, MetaInfoReader
from dotenv import load_dotenv

load_dotenv()

# 日本語フォントの設定（環境に応じて調整）
plt.rcParams['font.sans-serif'] = ['DejaVu Sans']
plt.rcParams['axes.unicode_minus'] = False

class TimeSeriesAnalyzer:
    """
    時系列データを分析するクラス
    """

    def __init__(self, stats_id):
        """
        初期化

        Parameters:
        -----------
        stats_id : str
            統計表ID
        """
        self.stats_id = stats_id
        self.reader = StatsDataReader()
        self.df = None

    def load_data(self, limit=None):
        """データを読み込む"""
        print(f"統計表ID {self.stats_id} のデータを読み込み中...")

        if limit:
            self.df = self.reader.read(statsDataId=self.stats_id, limit=limit)
        else:
            self.df = self.reader.read(statsDataId=self.stats_id)

        print(f"✅ {len(self.df)}件のデータを読み込みました")

        return self.df

    def prepare_timeseries(self, time_col, value_col, date_format=None):
        """
        時系列データを準備

        Parameters:
        -----------
        time_col : str
            時間カラム名
        value_col : str
            値カラム名
        date_format : str, optional
            日付フォーマット
        """
        if self.df is None:
            print("❌ データが読み込まれていません")
            return None

        # 時間カラムをインデックスに設定
        df_ts = self.df[[time_col, value_col]].copy()

        # 日付型に変換
        if date_format:
            df_ts[time_col] = pd.to_datetime(df_ts[time_col], format=date_format)
        else:
            df_ts[time_col] = pd.to_datetime(df_ts[time_col], errors='coerce')

        # インデックスに設定
        df_ts.set_index(time_col, inplace=True)

        # ソート
        df_ts.sort_index(inplace=True)

        print(f"✅ 時系列データ準備完了")
        print(f"   期間: {df_ts.index.min()} 〜 {df_ts.index.max()}")
        print(f"   データ数: {len(df_ts)}")

        return df_ts

    def plot_timeseries(self, df_ts, title="Time Series", ylabel="Value"):
        """
        時系列データをプロット

        Parameters:
        -----------
        df_ts : pandas.DataFrame
            時系列データ
        title : str
            グラフタイトル
        ylabel : str
            Y軸ラベル
        """
        plt.figure(figsize=(12, 6))

        plt.plot(df_ts.index, df_ts.iloc[:, 0], marker='o', linestyle='-', linewidth=2)

        plt.title(title, fontsize=16, fontweight='bold')
        plt.xlabel('Date', fontsize=12)
        plt.ylabel(ylabel, fontsize=12)
        plt.grid(True, alpha=0.3)
        plt.xticks(rotation=45)
        plt.tight_layout()

        # 保存
        filename = "timeseries_plot.png"
        plt.savefig(filename, dpi=300, bbox_inches='tight')
        print(f"✅ グラフを {filename} に保存しました")

        plt.show()

    def calculate_statistics(self, df_ts):
        """
        時系列統計量を計算

        Parameters:
        -----------
        df_ts : pandas.DataFrame
            時系列データ
        """
        print("\n【時系列統計量】")
        print("=" * 60)

        # 基本統計量
        print("\n基本統計量:")
        print(df_ts.describe())

        # 変化率（前期比）
        pct_change = df_ts.pct_change() * 100
        print(f"\n平均変化率（前期比）: {pct_change.mean().values[0]:.2f}%")
        print(f"最大増加率: {pct_change.max().values[0]:.2f}%")
        print(f"最大減少率: {pct_change.min().values[0]:.2f}%")

        # トレンド
        trend = (df_ts.iloc[-1] - df_ts.iloc[0]) / df_ts.iloc[0] * 100
        print(f"\n全期間のトレンド: {trend.values[0]:.2f}%")


# ===== 使用例 =====
# stats_id = "0003410379"
# analyzer = TimeSeriesAnalyzer(stats_id)
# analyzer.load_data(limit=1000)

# 実際のカラム名に置き換えてください
# df_ts = analyzer.prepare_timeseries(
#     time_col='時間コード',
#     value_col='値'
# )
# analyzer.plot_timeseries(df_ts, title="統計データの推移")
# analyzer.calculate_statistics(df_ts)
```

---

## 5.3 地域比較分析

### 5.3.1 複数地域の比較

```python
"""
複数地域のデータを比較分析
"""
import pandas as pd
import matplotlib.pyplot as plt
from jpy_datareader.estat import StatsDataReader, MetaInfoReader
from dotenv import load_dotenv

load_dotenv()

class RegionalComparisonAnalyzer:
    """
    地域比較分析を行うクラス
    """

    def __init__(self, stats_id):
        """初期化"""
        self.stats_id = stats_id
        self.meta_reader = MetaInfoReader()
        self.data_reader = StatsDataReader()
        self.df_meta = None
        self.area_data = {}

    def load_metadata(self):
        """メタ情報を読み込む"""
        self.df_meta = self.meta_reader.read(statsDataId=self.stats_id)
        print(f"✅ メタ情報を読み込みました")

    def get_area_data(self, area_names, limit=1000):
        """
        複数地域のデータを取得

        Parameters:
        -----------
        area_names : list
            地域名のリスト
        limit : int
            各地域の取得件数
        """
        if self.df_meta is None:
            self.load_metadata()

        # 地域分類を抽出
        df_area = self.df_meta[self.df_meta['@class'] == 'area']

        for area_name in area_names:
            print(f"\n{area_name}のデータを取得中...")

            # 地域コードを検索
            df_found = df_area[df_area['@name'].str.contains(area_name, na=False)]

            if len(df_found) == 0:
                print(f"  ❌ {area_name}が見つかりませんでした")
                continue

            area_code = df_found.iloc[0]['@code']

            # データを取得
            df_data = self.data_reader.read(
                statsDataId=self.stats_id,
                cdArea=area_code,
                limit=limit
            )

            self.area_data[area_name] = df_data
            print(f"  ✅ {len(df_data)}件取得")

    def compare_statistics(self, value_col):
        """
        地域間の統計量を比較

        Parameters:
        -----------
        value_col : str
            比較する値のカラム名
        """
        print("\n【地域比較統計量】")
        print("=" * 60)

        comparison = []

        for area_name, df in self.area_data.items():
            if value_col not in df.columns:
                print(f"⚠️ {area_name}: カラム'{value_col}'が見つかりません")
                continue

            stats = {
                '地域': area_name,
                'データ数': len(df),
                '平均': df[value_col].mean(),
                '中央値': df[value_col].median(),
                '最小': df[value_col].min(),
                '最大': df[value_col].max(),
                '標準偏差': df[value_col].std()
            }

            comparison.append(stats)

        # DataFrameに変換
        df_comparison = pd.DataFrame(comparison)

        print("\n")
        print(df_comparison.to_string(index=False))

        return df_comparison

    def plot_comparison(self, value_col, plot_type='bar'):
        """
        地域比較のグラフを作成

        Parameters:
        -----------
        value_col : str
            プロットする値のカラム名
        plot_type : str
            グラフの種類（'bar', 'box', 'line'）
        """
        if plot_type == 'bar':
            # 棒グラフ
            plt.figure(figsize=(12, 6))

            areas = []
            means = []

            for area_name, df in self.area_data.items():
                if value_col in df.columns:
                    areas.append(area_name)
                    means.append(df[value_col].mean())

            plt.bar(areas, means, color='steelblue', alpha=0.7)
            plt.title('Regional Comparison (Mean Values)', fontsize=16, fontweight='bold')
            plt.xlabel('Region', fontsize=12)
            plt.ylabel(f'Mean {value_col}', fontsize=12)
            plt.xticks(rotation=45, ha='right')
            plt.grid(axis='y', alpha=0.3)
            plt.tight_layout()

            filename = "regional_comparison_bar.png"
            plt.savefig(filename, dpi=300, bbox_inches='tight')
            print(f"✅ グラフを {filename} に保存しました")

            plt.show()

        elif plot_type == 'box':
            # 箱ひげ図
            plt.figure(figsize=(12, 6))

            data_list = []
            labels = []

            for area_name, df in self.area_data.items():
                if value_col in df.columns:
                    data_list.append(df[value_col].dropna())
                    labels.append(area_name)

            plt.boxplot(data_list, labels=labels)
            plt.title('Regional Comparison (Distribution)', fontsize=16, fontweight='bold')
            plt.xlabel('Region', fontsize=12)
            plt.ylabel(value_col, fontsize=12)
            plt.xticks(rotation=45, ha='right')
            plt.grid(axis='y', alpha=0.3)
            plt.tight_layout()

            filename = "regional_comparison_box.png"
            plt.savefig(filename, dpi=300, bbox_inches='tight')
            print(f"✅ グラフを {filename} に保存しました")

            plt.show()


# ===== 使用例 =====
# stats_id = "0003410379"
# analyzer = RegionalComparisonAnalyzer(stats_id)

# 地域リスト
# regions = ['東京都', '大阪府', '北海道', '福岡県']
# analyzer.get_area_data(regions, limit=500)

# 比較分析（実際のカラム名に置き換え）
# df_comparison = analyzer.compare_statistics(value_col='値')
# analyzer.plot_comparison(value_col='値', plot_type='bar')
```

---

## 5.4 実践プロジェクト: 都道府県別人口動態分析

### 5.4.1 プロジェクト全体のコード

```python
"""
実践プロジェクト: 都道府県別人口動態分析
"""
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from jpy_datareader.estat import StatsDataReader, MetaInfoReader
from dotenv import load_dotenv
import numpy as np

load_dotenv()

class PopulationAnalysisProject:
    """
    人口動態分析プロジェクト
    """

    def __init__(self, stats_id):
        """初期化"""
        self.stats_id = stats_id
        self.meta_reader = MetaInfoReader()
        self.data_reader = StatsDataReader()
        self.df_data = None
        self.df_meta = None

    def step1_load_data(self, limit=None):
        """ステップ1: データを読み込む"""
        print("=" * 60)
        print("【ステップ1】データの読み込み")
        print("=" * 60)

        # メタ情報を読み込む
        self.df_meta = self.meta_reader.read(statsDataId=self.stats_id)
        print(f"✅ メタ情報: {len(self.df_meta)}件")

        # データを読み込む
        if limit:
            self.df_data = self.data_reader.read(
                statsDataId=self.stats_id,
                limit=limit
            )
        else:
            self.df_data = self.data_reader.read(statsDataId=self.stats_id)

        print(f"✅ データ: {len(self.df_data)}件")
        print(f"   カラム数: {len(self.df_data.columns)}")

        return self.df_data

    def step2_explore_data(self):
        """ステップ2: データを探索する"""
        print("\n" + "=" * 60)
        print("【ステップ2】データの探索")
        print("=" * 60)

        # データ型を確認
        print("\n【データ型】")
        print(self.df_data.dtypes)

        # 欠損値を確認
        print("\n【欠損値】")
        missing = self.df_data.isnull().sum()
        missing = missing[missing > 0]
        if len(missing) > 0:
            print(missing)
        else:
            print("欠損値なし")

        # 基本統計量
        print("\n【基本統計量】")
        numeric_cols = self.df_data.select_dtypes(include=[np.number]).columns
        if len(numeric_cols) > 0:
            print(self.df_data[numeric_cols].describe())

    def step3_clean_data(self):
        """ステップ3: データをクリーニングする"""
        print("\n" + "=" * 60)
        print("【ステップ3】データのクリーニング")
        print("=" * 60)

        # 元のデータ件数
        original_count = len(self.df_data)

        # 欠損値を削除
        self.df_data = self.df_data.dropna()

        # 重複を削除
        self.df_data = self.df_data.drop_duplicates()

        cleaned_count = len(self.df_data)
        removed = original_count - cleaned_count

        print(f"元データ: {original_count}件")
        print(f"削除: {removed}件")
        print(f"クリーン後: {cleaned_count}件")

        return self.df_data

    def step4_analyze(self, group_col, value_col):
        """
        ステップ4: データを分析する

        Parameters:
        -----------
        group_col : str
            グループ化するカラム
        value_col : str
            分析する値のカラム
        """
        print("\n" + "=" * 60)
        print("【ステップ4】データの分析")
        print("=" * 60)

        if group_col not in self.df_data.columns:
            print(f"❌ カラム'{group_col}'が見つかりません")
            return None

        if value_col not in self.df_data.columns:
            print(f"❌ カラム'{value_col}'が見つかりません")
            return None

        # グループ別集計
        df_grouped = self.df_data.groupby(group_col)[value_col].agg([
            ('件数', 'count'),
            ('合計', 'sum'),
            ('平均', 'mean'),
            ('中央値', 'median'),
            ('最小', 'min'),
            ('最大', 'max'),
            ('標準偏差', 'std')
        ]).reset_index()

        # ソート（平均値の降順）
        df_grouped = df_grouped.sort_values('平均', ascending=False)

        print(f"\n【{group_col}別の統計量】")
        print(df_grouped.to_string(index=False))

        return df_grouped

    def step5_visualize(self, df_grouped, group_col, value_col='平均'):
        """
        ステップ5: 結果を可視化する

        Parameters:
        -----------
        df_grouped : pandas.DataFrame
            集計データ
        group_col : str
            グループカラム名
        value_col : str
            表示する値のカラム名
        """
        print("\n" + "=" * 60)
        print("【ステップ5】データの可視化")
        print("=" * 60)

        # 上位10件のみ表示
        df_top = df_grouped.head(10)

        # 棒グラフ
        plt.figure(figsize=(12, 6))

        plt.bar(range(len(df_top)), df_top[value_col], color='steelblue', alpha=0.7)
        plt.xticks(range(len(df_top)), df_top[group_col], rotation=45, ha='right')
        plt.xlabel(group_col, fontsize=12)
        plt.ylabel(value_col, fontsize=12)
        plt.title(f'Top 10 {group_col} by {value_col}', fontsize=16, fontweight='bold')
        plt.grid(axis='y', alpha=0.3)
        plt.tight_layout()

        filename = "analysis_result.png"
        plt.savefig(filename, dpi=300, bbox_inches='tight')
        print(f"✅ グラフを {filename} に保存しました")

        plt.show()

    def step6_export(self, df_grouped, filename="analysis_result.csv"):
        """
        ステップ6: 結果をエクスポートする

        Parameters:
        -----------
        df_grouped : pandas.DataFrame
            集計データ
        filename : str
            出力ファイル名
        """
        print("\n" + "=" * 60)
        print("【ステップ6】結果のエクスポート")
        print("=" * 60)

        # CSVに保存
        df_grouped.to_csv(filename, index=False, encoding='utf-8-sig')
        print(f"✅ 分析結果を {filename} に保存しました")

        # レポートも作成
        report_filename = filename.replace('.csv', '_report.txt')

        with open(report_filename, 'w', encoding='utf-8') as f:
            f.write("=" * 60 + "\n")
            f.write("データ分析レポート\n")
            f.write("=" * 60 + "\n\n")
            f.write(f"統計表ID: {self.stats_id}\n")
            f.write(f"データ件数: {len(self.df_data)}\n")
            f.write(f"グループ数: {len(df_grouped)}\n\n")
            f.write("=" * 60 + "\n")
            f.write("集計結果（上位10件）\n")
            f.write("=" * 60 + "\n")
            f.write(df_grouped.head(10).to_string(index=False))

        print(f"✅ レポートを {report_filename} に保存しました")


# ===== プロジェクトの実行 =====
# stats_id = "0003410379"  # 実際の統計表IDに置き換え
# project = PopulationAnalysisProject(stats_id)

# ステップ1: データ読み込み
# project.step1_load_data(limit=5000)

# ステップ2: データ探索
# project.step2_explore_data()

# ステップ3: データクリーニング
# project.step3_clean_data()

# ステップ4: 分析（実際のカラム名に置き換え）
# df_result = project.step4_analyze(group_col='地域名', value_col='値')

# ステップ5: 可視化
# project.step5_visualize(df_result, group_col='地域名')

# ステップ6: エクスポート
# project.step6_export(df_result)

print("\n" + "=" * 60)
print("プロジェクト完了！")
print("=" * 60)
```

---

## 📝 練習問題

### 問題1: 時系列分析

**課題**: 特定の地域（例: 東京都）の時系列データを取得し、前年比の変化率を計算して可視化してください。

### 問題2: 地域ランキング

**課題**: すべての都道府県のデータを取得し、特定の指標（例: 人口）でランキングを作成してください。上位10都道府県と下位10都道府県を表示してください。

### 問題3: 相関分析

**課題**: 2つの異なる統計表から関連するデータを取得し、相関係数を計算してください。散布図も作成してください。

### 問題4: 地域グループ分析

**課題**: 都道府県を地方ブロック（例: 関東、関西など）にグループ化し、グループ別の統計量を比較してください。

### 問題5: 総合プロジェクト

**課題**: 以下の要件を満たす分析プロジェクトを作成してください：
1. 複数年のデータを取得
2. 地域別・年別の集計
3. トレンド分析
4. 可視化（グラフ3種類以上）
5. レポート出力

---

## 📖 模範解答

### 解答1: 時系列分析

```python
"""
問題1の模範解答: 時系列分析と前年比計算
"""
import pandas as pd
import matplotlib.pyplot as plt
from jpy_datareader.estat import StatsDataReader, MetaInfoReader
from dotenv import load_dotenv

load_dotenv()

print("【時系列分析: 東京都の前年比変化】")
print("=" * 60)

stats_id = "0003410379"  # 実際のIDに置き換え

# メタ情報から東京都のコードを取得
meta_reader = MetaInfoReader()
df_meta = meta_reader.read(statsDataId=stats_id)

df_area = df_meta[df_meta['@class'] == 'area']
df_tokyo = df_area[df_area['@name'].str.contains('東京都', na=False)]

if len(df_tokyo) > 0:
    tokyo_code = df_tokyo.iloc[0]['@code']
    print(f"✅ 東京都コード: {tokyo_code}")

    # データを取得
    data_reader = StatsDataReader()
    df_data = data_reader.read(
        statsDataId=stats_id,
        cdArea=tokyo_code,
        limit=1000
    )

    print(f"✅ {len(df_data)}件のデータを取得")

    # 時系列データに変換（実際のカラム名に置き換え）
    # df_data['年月'] = pd.to_datetime(df_data['時間コード'])
    # df_data = df_data.sort_values('年月')

    # 前年比を計算
    # df_data['前年比'] = df_data['値'].pct_change(12) * 100  # 12ヶ月前との比較

    # 可視化
    # fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 10))

    # 元の値
    # ax1.plot(df_data['年月'], df_data['値'], marker='o', linewidth=2)
    # ax1.set_title('Tokyo: Time Series', fontsize=14, fontweight='bold')
    # ax1.set_ylabel('Value', fontsize=12)
    # ax1.grid(True, alpha=0.3)

    # 前年比
    # ax2.plot(df_data['年月'], df_data['前年比'], marker='o', color='red', linewidth=2)
    # ax2.axhline(y=0, color='black', linestyle='--', alpha=0.5)
    # ax2.set_title('Year-over-Year Change Rate', fontsize=14, fontweight='bold')
    # ax2.set_xlabel('Date', fontsize=12)
    # ax2.set_ylabel('Change Rate (%)', fontsize=12)
    # ax2.grid(True, alpha=0.3)

    # plt.tight_layout()
    # plt.savefig('tokyo_timeseries_yoy.png', dpi=300)
    # print("✅ グラフを保存しました")
```

### 解答2〜5は同様の構造で実装できます。

---

## 🎯 まとめ

この章では以下を学びました：

✅ 複数統計表の結合テクニック
✅ 時系列データの分析手法
✅ 地域比較分析の実践
✅ データ可視化の技術
✅ 実践的な分析プロジェクトの構築

---

## 📚 次のステップ

**[第6章: 上級編 - エラーハンドリングとベストプラクティス](./chapter06_best_practices.md)**

- エラーハンドリングの詳細
- パフォーマンス最適化
- セキュリティ対策
- 本番環境での運用

---

お疲れ様でした！
