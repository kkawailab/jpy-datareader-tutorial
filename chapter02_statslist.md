# 第2章: 中級編 - StatsListReaderで統計表を探す

## 📚 この章で学ぶこと

- StatsListReaderクラスの詳細な使い方
- 高度な検索テクニック
- フィルタリングとソート
- 複数条件での検索
- 検索結果の効率的な活用

**対象レベル**: 中級者
**所要時間**: 約45分
**前提知識**: 第1章の内容、pandasの基本操作

---

## 2.1 StatsListReaderとは

### 2.1.1 概要

`StatsListReader`は、e-Statの統計表を検索するためのクラスです。第1章で学んだ`get_data_estat_statslist()`関数よりも詳細な制御が可能です。

### 2.1.2 基本的な違い

| 機能 | get_data_estat_statslist() | StatsListReader |
|------|---------------------------|-----------------|
| 使いやすさ | ⭐⭐⭐ | ⭐⭐ |
| カスタマイズ性 | ⭐ | ⭐⭐⭐ |
| 詳細な制御 | ❌ | ✅ |
| パラメータ設定 | シンプル | 詳細 |

---

## 2.2 基本的な使い方

### 2.2.1 インポートと初期化

```python
"""
StatsListReaderの基本的な使い方
"""
import os
from dotenv import load_dotenv
from jpy_datareader.estat import StatsListReader
import pandas as pd

# 環境変数を読み込む
load_dotenv()

# StatsListReaderのインスタンスを作成
# app_idは環境変数から自動取得されるため、通常は省略可能
reader = StatsListReader()

# または、明示的にAPIキーを指定することも可能
# api_key = os.getenv('ESTAT_APP_ID')
# reader = StatsListReader(app_id=api_key)

print("✅ StatsListReaderを初期化しました")
```

### 2.2.2 基本的な検索

```python
"""
基本的な統計表検索
"""
from jpy_datareader.estat import StatsListReader

# Readerのインスタンスを作成
reader = StatsListReader()

# キーワード検索を実行
# read()メソッドでデータを取得
df = reader.read(
    searchWord="人口",  # 検索キーワード
    limit=10            # 取得件数
)

# 結果を表示
print(f"検索結果: {len(df)}件")
print("\n統計表一覧:")
print(df[['@id', 'STAT_NAME', 'TITLE']].head())
```

---

## 2.3 詳細なパラメータ設定

### 2.3.1 利用可能なパラメータ一覧

StatsListReaderの`read()`メソッドには、多くのパラメータがあります：

```python
"""
StatsListReaderの全パラメータ
"""
from jpy_datareader.estat import StatsListReader

reader = StatsListReader()

df = reader.read(
    # 検索条件
    searchWord="人口",           # 検索キーワード
    searchKind=2,                # 検索方法 (1: AND, 2: OR, 3: NOT)

    # データ取得設定
    limit=100,                   # 取得件数（最大100,000）
    startPosition=1,             # 取得開始位置

    # 日付フィルタ
    updatedDate=None,            # 更新日時（YYYY-MM-DD形式）

    # 統計分類コード
    statsCode=None,              # 政府統計コード
    surveyYears=None,            # 調査年月

    # その他
    lang="J"                     # 言語 (J: 日本語, E: 英語)
)

print(df.head())
```

### 2.3.2 パラメータの詳細説明

```python
"""
パラメータの使用例と説明
"""
from jpy_datareader.estat import StatsListReader
from datetime import datetime, timedelta

reader = StatsListReader()

# 例1: 検索方法の指定
# searchKind = 1 (AND検索)
print("【例1】AND検索 - 人口 AND 推計")
df_and = reader.read(
    searchWord="人口 推計",  # スペース区切りでAND検索
    searchKind=1,            # AND検索を指定
    limit=5
)
print(f"結果件数: {len(df_and)}")
print(df_and[['TITLE']].head())

# 例2: OR検索
print("\n【例2】OR検索 - 人口 OR GDP")
df_or = reader.read(
    searchWord="人口 GDP",   # スペース区切りでOR検索
    searchKind=2,            # OR検索を指定
    limit=5
)
print(f"結果件数: {len(df_or)}")

# 例3: 取得位置の指定（ページネーション）
print("\n【例3】ページネーション")
# 最初の10件
df_page1 = reader.read(
    searchWord="家計",
    limit=10,
    startPosition=1          # 1件目から
)
print(f"1ページ目: {len(df_page1)}件")

# 次の10件
df_page2 = reader.read(
    searchWord="家計",
    limit=10,
    startPosition=11         # 11件目から
)
print(f"2ページ目: {len(df_page2)}件")

# 例4: 更新日でフィルタ
print("\n【例4】更新日でフィルタ")
# 2024年1月1日以降に更新された統計
df_recent = reader.read(
    searchWord="賃金",
    updatedDate="2024-01-01",  # この日以降に更新されたもの
    limit=5
)
print(f"2024年以降更新: {len(df_recent)}件")
```

---

## 2.4 高度な検索テクニック

### 2.4.1 複合条件での検索

```python
"""
複合条件での高度な検索
"""
from jpy_datareader.estat import StatsListReader
import pandas as pd

reader = StatsListReader()

def advanced_search(
    keywords,
    stats_code=None,
    updated_after=None,
    max_results=50
):
    """
    複合条件で統計表を検索する

    Parameters:
    -----------
    keywords : str
        検索キーワード
    stats_code : str, optional
        政府統計コード
    updated_after : str, optional
        更新日の下限（YYYY-MM-DD形式）
    max_results : int
        最大取得件数

    Returns:
    --------
    pandas.DataFrame
        検索結果
    """
    try:
        # 検索パラメータを構築
        params = {
            'searchWord': keywords,
            'limit': max_results,
            'searchKind': 1  # AND検索
        }

        # オプショナルパラメータを追加
        if stats_code:
            params['statsCode'] = stats_code

        if updated_after:
            params['updatedDate'] = updated_after

        # 検索を実行
        df = reader.read(**params)

        print(f"✅ {len(df)}件の統計表が見つかりました")

        return df

    except Exception as e:
        print(f"❌ エラー: {e}")
        return None

# 使用例1: キーワードのみ
print("【使用例1】キーワード検索")
print("=" * 60)
df1 = advanced_search(keywords="人口 推計")
if df1 is not None:
    print(df1[['STAT_NAME', 'TITLE']].head())

# 使用例2: 政府統計コードも指定
print("\n【使用例2】政府統計コード指定")
print("=" * 60)
# 00200524 = 国勢調査
df2 = advanced_search(
    keywords="人口",
    stats_code="00200524"
)
if df2 is not None:
    print(df2[['STAT_NAME', 'TITLE']].head())

# 使用例3: 更新日も指定
print("\n【使用例3】更新日指定")
print("=" * 60)
df3 = advanced_search(
    keywords="GDP",
    updated_after="2024-01-01"
)
if df3 is not None:
    print(df3[['STAT_NAME', 'TITLE', 'UPDATED_DATE']].head())
```

### 2.4.2 検索結果のフィルタリング

```python
"""
検索結果をさらにフィルタリングする
"""
from jpy_datareader.estat import StatsListReader
import pandas as pd

reader = StatsListReader()

# 幅広く検索
df = reader.read(searchWord="統計", limit=100)

print(f"初期検索結果: {len(df)}件")

# ===== フィルタ1: 特定の統計名だけを抽出 =====
print("\n【フィルタ1】特定の統計名を抽出")
df_filtered = df[df['STAT_NAME'].str.contains('国勢調査', na=False)]
print(f"国勢調査のみ: {len(df_filtered)}件")
print(df_filtered[['STAT_NAME', 'TITLE']].head())

# ===== フィルタ2: タイトルに特定のキーワードを含む =====
print("\n【フィルタ2】タイトルフィルタ")
df_title = df[df['TITLE'].str.contains('都道府県', na=False)]
print(f"「都道府県」を含む: {len(df_title)}件")
print(df_title[['STAT_NAME', 'TITLE']].head())

# ===== フィルタ3: 最近更新されたもの =====
print("\n【フィルタ3】更新日でフィルタ")
# UPDATED_DATEを日付型に変換
df['UPDATED_DATE'] = pd.to_datetime(df['UPDATED_DATE'])

# 2024年以降に更新されたもの
recent_date = pd.to_datetime('2024-01-01')
df_recent = df[df['UPDATED_DATE'] >= recent_date]
print(f"2024年以降: {len(df_recent)}件")

# ===== フィルタ4: 複数条件の組み合わせ =====
print("\n【フィルタ4】複合条件")
df_complex = df[
    (df['STAT_NAME'].str.contains('調査', na=False)) &
    (df['TITLE'].str.contains('月次', na=False))
]
print(f"「調査」かつ「月次」: {len(df_complex)}件")
print(df_complex[['STAT_NAME', 'TITLE']].head())
```

---

## 2.5 検索結果のソートと整理

### 2.5.1 ソート機能

```python
"""
検索結果のソートと整理
"""
from jpy_datareader.estat import StatsListReader
import pandas as pd

reader = StatsListReader()

# 検索を実行
df = reader.read(searchWord="経済", limit=50)

# ===== ソート1: 更新日の新しい順 =====
print("【ソート1】更新日の新しい順")
df['UPDATED_DATE'] = pd.to_datetime(df['UPDATED_DATE'])
df_sorted = df.sort_values('UPDATED_DATE', ascending=False)

print("最新の5件:")
for idx, row in df_sorted.head().iterrows():
    print(f"  {row['UPDATED_DATE'].date()} | {row['TITLE'][:50]}...")

# ===== ソート2: 統計名のアルファベット順 =====
print("\n【ソート2】統計名のアルファベット順")
df_sorted_name = df.sort_values('STAT_NAME')

print("統計名順の5件:")
for idx, row in df_sorted_name.head().iterrows():
    print(f"  {row['STAT_NAME']} | {row['TITLE'][:40]}...")

# ===== グループ化: 統計名ごとに集計 =====
print("\n【グループ化】統計名ごとの件数")
stat_counts = df['STAT_NAME'].value_counts()

print("統計名別の件数（上位10件）:")
for stat_name, count in stat_counts.head(10).items():
    print(f"  {stat_name}: {count}件")
```

### 2.5.2 検索結果の保存と再利用

```python
"""
検索結果を保存して再利用する
"""
from jpy_datareader.estat import StatsListReader
import pandas as pd
import json

reader = StatsListReader()

# ===== 検索結果をCSVに保存 =====
def save_search_results(keywords, filename):
    """
    検索結果をCSVに保存

    Parameters:
    -----------
    keywords : str
        検索キーワード
    filename : str
        保存先ファイル名
    """
    # 検索を実行
    df = reader.read(searchWord=keywords, limit=100)

    # CSVに保存
    df.to_csv(filename, index=False, encoding='utf-8-sig')

    print(f"✅ {len(df)}件の検索結果を {filename} に保存しました")

    # 主要カラムのみのファイルも作成
    summary_filename = filename.replace('.csv', '_summary.csv')
    df[['@id', 'STAT_NAME', 'TITLE', 'UPDATED_DATE']].to_csv(
        summary_filename,
        index=False,
        encoding='utf-8-sig'
    )
    print(f"✅ サマリーを {summary_filename} に保存しました")

    return df

# 使用例
print("【検索と保存】")
df_saved = save_search_results("人口", "population_stats.csv")

# ===== 保存したデータを読み込む =====
def load_saved_results(filename):
    """
    保存した検索結果を読み込む

    Parameters:
    -----------
    filename : str
        読み込むファイル名

    Returns:
    --------
    pandas.DataFrame
        読み込んだデータ
    """
    try:
        df = pd.read_csv(filename, encoding='utf-8-sig')
        print(f"✅ {filename} から {len(df)}件のデータを読み込みました")
        return df

    except FileNotFoundError:
        print(f"❌ ファイルが見つかりません: {filename}")
        return None

# 使用例
print("\n【読み込み】")
df_loaded = load_saved_results("population_stats.csv")
if df_loaded is not None:
    print(df_loaded.head())
```

---

## 2.6 実践: 統計表カタログの作成

実際のプロジェクトで使える統計表カタログを作成してみましょう。

```python
"""
実践: 統計表カタログの作成
複数のカテゴリで検索し、統合されたカタログを作成
"""
from jpy_datareader.estat import StatsListReader
import pandas as pd
from datetime import datetime

class StatsCatalog:
    """
    統計表カタログを管理するクラス
    """

    def __init__(self):
        """初期化"""
        self.reader = StatsListReader()
        self.catalog = pd.DataFrame()

    def add_category(self, category_name, keywords, limit=50):
        """
        カテゴリを追加

        Parameters:
        -----------
        category_name : str
            カテゴリ名
        keywords : str
            検索キーワード
        limit : int
            取得件数
        """
        print(f"\n【{category_name}】を検索中...")

        # 検索を実行
        df = self.reader.read(searchWord=keywords, limit=limit)

        # カテゴリ列を追加
        df['CATEGORY'] = category_name

        # カタログに追加
        self.catalog = pd.concat([self.catalog, df], ignore_index=True)

        print(f"  ✅ {len(df)}件追加しました")

    def remove_duplicates(self):
        """重複を削除"""
        before_count = len(self.catalog)

        # @idで重複を削除
        self.catalog = self.catalog.drop_duplicates(subset=['@id'])

        after_count = len(self.catalog)
        removed = before_count - after_count

        print(f"\n✅ 重複を削除: {removed}件削除、{after_count}件残存")

    def get_summary(self):
        """カタログのサマリーを表示"""
        print("\n" + "=" * 60)
        print("【統計表カタログ サマリー】")
        print("=" * 60)

        # 総件数
        print(f"\n総統計表数: {len(self.catalog)}件")

        # カテゴリ別件数
        print("\nカテゴリ別件数:")
        category_counts = self.catalog['CATEGORY'].value_counts()
        for cat, count in category_counts.items():
            print(f"  {cat}: {count}件")

        # 統計名別件数（上位10件）
        print("\n統計名別件数（上位10件）:")
        stat_counts = self.catalog['STAT_NAME'].value_counts().head(10)
        for stat, count in stat_counts.items():
            print(f"  {stat}: {count}件")

    def save_catalog(self, filename="stats_catalog.csv"):
        """カタログを保存"""
        # 保存前にソート
        self.catalog = self.catalog.sort_values(
            ['CATEGORY', 'STAT_NAME'],
            ascending=True
        )

        # 保存
        self.catalog.to_csv(filename, index=False, encoding='utf-8-sig')
        print(f"\n✅ カタログを {filename} に保存しました")

        # サマリー版も作成
        summary_cols = ['@id', 'CATEGORY', 'STAT_NAME', 'TITLE', 'UPDATED_DATE']
        summary_file = filename.replace('.csv', '_summary.csv')
        self.catalog[summary_cols].to_csv(
            summary_file,
            index=False,
            encoding='utf-8-sig'
        )
        print(f"✅ サマリーを {summary_file} に保存しました")

    def search_in_catalog(self, keyword):
        """
        カタログ内を検索

        Parameters:
        -----------
        keyword : str
            検索キーワード
        """
        # タイトルまたは統計名にキーワードを含むものを検索
        results = self.catalog[
            (self.catalog['TITLE'].str.contains(keyword, na=False)) |
            (self.catalog['STAT_NAME'].str.contains(keyword, na=False))
        ]

        print(f"\n【カタログ検索】キーワード: '{keyword}'")
        print(f"検索結果: {len(results)}件")

        if len(results) > 0:
            print("\n検索結果:")
            for idx, row in results.head(10).iterrows():
                print(f"  [{row['CATEGORY']}] {row['STAT_NAME']}")
                print(f"    {row['TITLE'][:60]}...")

        return results


# ===== カタログの作成 =====
print("=" * 60)
print("統計表カタログの作成を開始")
print("=" * 60)

# カタログのインスタンスを作成
catalog = StatsCatalog()

# カテゴリを追加
catalog.add_category("人口・世帯", "人口 世帯", limit=30)
catalog.add_category("経済・産業", "GDP 経済 産業", limit=30)
catalog.add_category("労働・賃金", "賃金 雇用 労働", limit=30)
catalog.add_category("家計・消費", "家計 消費 支出", limit=30)
catalog.add_category("物価", "物価 CPI 消費者物価", limit=20)

# 重複を削除
catalog.remove_duplicates()

# サマリーを表示
catalog.get_summary()

# カタログを保存
catalog.save_catalog()

# カタログ内を検索してみる
print("\n" + "=" * 60)
print("カタログ検索のテスト")
print("=" * 60)
catalog.search_in_catalog("都道府県")
```

---

## 📝 練習問題

### 問題1: 基本的なStatsListReaderの使用

**課題**: StatsListReaderを使って、「消費」というキーワードでAND検索を行い、最新の10件を更新日の新しい順に表示してください。

<details>
<summary>ヒント</summary>

- `StatsListReader()`でインスタンス作成
- `searchKind=1`でAND検索
- `sort_values('UPDATED_DATE', ascending=False)`でソート

</details>

---

### 問題2: ページネーション機能

**課題**: 「経済」というキーワードで検索し、21件目から30件目のデータを取得してください。

<details>
<summary>ヒント</summary>

- `startPosition=21`を指定
- `limit=10`で10件取得

</details>

---

### 問題3: フィルタリングとグループ化

**課題**: 「調査」というキーワードで50件取得し、統計名ごとにグループ化して件数を表示してください。その後、最も件数が多い統計名の詳細を表示してください。

<details>
<summary>ヒント</summary>

- `value_counts()`でグループ化
- `idxmax()`で最大値のインデックスを取得
- フィルタリングで該当データを抽出

</details>

---

### 問題4: カスタム検索関数

**課題**: キーワード、政府統計コード、更新日を指定して検索できる関数を作成してください。すべてのパラメータはオプショナルとし、指定された条件のみで検索するようにしてください。

<details>
<summary>ヒント</summary>

- 関数の引数をすべて`None`をデフォルト値に
- 辞書でパラメータを構築
- `**kwargs`で動的にパラメータを渡す

</details>

---

### 問題5: 検索結果の比較

**課題**: 「人口」と「GDP」の2つのキーワードでそれぞれ検索し、結果の統計名の種類を比較してください。どちらにも含まれる統計名を表示してください。

<details>
<summary>ヒント</summary>

- 2回検索を実行
- `set()`で集合に変換
- `&`演算子で共通要素を取得

</details>

---

## 📖 模範解答

### 解答1: 基本的なStatsListReaderの使用

```python
"""
問題1の模範解答: AND検索と更新日ソート
"""
from jpy_datareader.estat import StatsListReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

# StatsListReaderのインスタンスを作成
reader = StatsListReader()

# 「消費」でAND検索
df = reader.read(
    searchWord="消費",    # 検索キーワード
    searchKind=1,         # AND検索
    limit=10              # 10件取得
)

print(f"検索結果: {len(df)}件")

# 更新日を日付型に変換
df['UPDATED_DATE'] = pd.to_datetime(df['UPDATED_DATE'])

# 更新日の新しい順にソート
df_sorted = df.sort_values('UPDATED_DATE', ascending=False)

# 結果を表示
print("\n【最新の10件】")
print("-" * 80)
for idx, row in df_sorted.iterrows():
    print(f"更新日: {row['UPDATED_DATE'].date()}")
    print(f"統計名: {row['STAT_NAME']}")
    print(f"タイトル: {row['TITLE'][:60]}...")
    print("-" * 80)
```

---

### 解答2: ページネーション機能

```python
"""
問題2の模範解答: ページネーション
"""
from jpy_datareader.estat import StatsListReader
from dotenv import load_dotenv

load_dotenv()

reader = StatsListReader()

# 21件目から30件目を取得
df = reader.read(
    searchWord="経済",
    startPosition=21,     # 21件目から開始
    limit=10              # 10件取得
)

print(f"取得データ: {len(df)}件（21〜30件目）")

# 結果を表示
print("\n【21〜30件目】")
for i, (idx, row) in enumerate(df.iterrows(), start=21):
    print(f"\n{i}. ID: {row['@id']}")
    print(f"   統計名: {row['STAT_NAME']}")
    print(f"   タイトル: {row['TITLE'][:50]}...")

# 補足: 最初の10件と比較
print("\n【比較用】最初の10件も取得")
df_first = reader.read(
    searchWord="経済",
    startPosition=1,
    limit=10
)
print(f"1〜10件目: {len(df_first)}件")

# IDが重複していないことを確認
common_ids = set(df['@id']).intersection(set(df_first['@id']))
print(f"\n重複チェック: {len(common_ids)}件の重複（0件が正常）")
```

---

### 解答3: フィルタリングとグループ化

```python
"""
問題3の模範解答: グループ化と分析
"""
from jpy_datareader.estat import StatsListReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

reader = StatsListReader()

# 「調査」で検索
df = reader.read(searchWord="調査", limit=50)

print(f"検索結果: {len(df)}件")

# ===== ステップ1: 統計名ごとにグループ化 =====
print("\n【統計名別の件数】")
print("=" * 60)

stat_counts = df['STAT_NAME'].value_counts()
print(stat_counts)

# ===== ステップ2: 最も件数が多い統計名を特定 =====
most_common_stat = stat_counts.idxmax()
max_count = stat_counts.max()

print(f"\n最も件数が多い統計名: {most_common_stat} ({max_count}件)")

# ===== ステップ3: その統計名の詳細を表示 =====
print(f"\n【{most_common_stat}の詳細】")
print("=" * 60)

df_filtered = df[df['STAT_NAME'] == most_common_stat]

for idx, row in df_filtered.iterrows():
    print(f"\nID: {row['@id']}")
    print(f"タイトル: {row['TITLE']}")
    print(f"更新日: {row['UPDATED_DATE']}")
    print("-" * 60)

# ===== 追加分析: 統計名の分布を可視化 =====
print("\n【統計名分布（上位10件）】")
print("=" * 60)

top_10_stats = stat_counts.head(10)
for stat_name, count in top_10_stats.items():
    # 棒グラフ風の表示
    bar = "■" * count
    print(f"{stat_name:30} | {bar} ({count})")
```

---

### 解答4: カスタム検索関数

```python
"""
問題4の模範解答: 柔軟な検索関数
"""
from jpy_datareader.estat import StatsListReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

def flexible_search(
    keywords=None,
    stats_code=None,
    updated_after=None,
    search_kind=2,
    limit=50,
    sort_by='UPDATED_DATE',
    ascending=False
):
    """
    柔軟な検索関数

    Parameters:
    -----------
    keywords : str, optional
        検索キーワード
    stats_code : str, optional
        政府統計コード
    updated_after : str, optional
        更新日の下限（YYYY-MM-DD）
    search_kind : int
        検索方法（1: AND, 2: OR）
    limit : int
        取得件数
    sort_by : str
        ソートカラム
    ascending : bool
        昇順かどうか

    Returns:
    --------
    pandas.DataFrame
        検索結果
    """
    reader = StatsListReader()

    # パラメータを構築
    params = {
        'limit': limit,
        'searchKind': search_kind
    }

    # オプショナルパラメータを追加
    if keywords:
        params['searchWord'] = keywords

    if stats_code:
        params['statsCode'] = stats_code

    if updated_after:
        params['updatedDate'] = updated_after

    # キーワードが指定されていない場合はエラー
    if not keywords and not stats_code:
        print("❌ キーワードまたは統計コードを指定してください")
        return None

    try:
        # 検索を実行
        df = reader.read(**params)

        print(f"✅ {len(df)}件の統計表が見つかりました")

        # ソート
        if sort_by in df.columns:
            # 日付カラムの場合は変換
            if 'DATE' in sort_by:
                df[sort_by] = pd.to_datetime(df[sort_by])

            df = df.sort_values(sort_by, ascending=ascending)

        return df

    except Exception as e:
        print(f"❌ エラー: {e}")
        return None


# ===== テストケース =====

# テスト1: キーワードのみ
print("【テスト1】キーワードのみ")
print("=" * 60)
df1 = flexible_search(keywords="人口")
if df1 is not None:
    print(df1[['STAT_NAME', 'TITLE']].head())

# テスト2: キーワード + 更新日
print("\n【テスト2】キーワード + 更新日")
print("=" * 60)
df2 = flexible_search(
    keywords="GDP",
    updated_after="2024-01-01"
)
if df2 is not None:
    print(df2[['STAT_NAME', 'TITLE', 'UPDATED_DATE']].head())

# テスト3: 統計コードのみ
print("\n【テスト3】統計コードのみ")
print("=" * 60)
df3 = flexible_search(stats_code="00200524")  # 国勢調査
if df3 is not None:
    print(df3[['STAT_NAME', 'TITLE']].head())

# テスト4: すべてのパラメータ
print("\n【テスト4】全パラメータ指定")
print("=" * 60)
df4 = flexible_search(
    keywords="人口 推計",
    stats_code="00200524",
    updated_after="2020-01-01",
    search_kind=1,  # AND検索
    limit=20,
    sort_by='TITLE',
    ascending=True
)
if df4 is not None:
    print(df4[['STAT_NAME', 'TITLE']].head())

# テスト5: パラメータなし（エラーケース）
print("\n【テスト5】パラメータなし（エラー）")
print("=" * 60)
df5 = flexible_search()
```

---

### 解答5: 検索結果の比較

```python
"""
問題5の模範解答: 検索結果の比較分析
"""
from jpy_datareader.estat import StatsListReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

reader = StatsListReader()

# ===== ステップ1: 2つのキーワードで検索 =====
print("【ステップ1】検索実行")
print("=" * 60)

# 「人口」で検索
df_population = reader.read(searchWord="人口", limit=50)
print(f"「人口」の検索結果: {len(df_population)}件")

# 「GDP」で検索
df_gdp = reader.read(searchWord="GDP", limit=50)
print(f"「GDP」の検索結果: {len(df_gdp)}件")

# ===== ステップ2: 統計名の種類を抽出 =====
print("\n【ステップ2】統計名の抽出")
print("=" * 60)

# 統計名をセットに変換
stats_population = set(df_population['STAT_NAME'].unique())
stats_gdp = set(df_gdp['STAT_NAME'].unique())

print(f"「人口」の統計名の種類: {len(stats_population)}種類")
print(f"「GDP」の統計名の種類: {len(stats_gdp)}種類")

# ===== ステップ3: 共通する統計名を抽出 =====
print("\n【ステップ3】共通統計名の抽出")
print("=" * 60)

# 集合の積（共通要素）を計算
common_stats = stats_population & stats_gdp

print(f"共通する統計名: {len(common_stats)}種類")

if len(common_stats) > 0:
    print("\n共通統計名一覧:")
    for stat in sorted(common_stats):
        print(f"  - {stat}")
else:
    print("\n共通する統計名はありませんでした")

# ===== ステップ4: 各キーワード固有の統計名 =====
print("\n【ステップ4】固有の統計名")
print("=" * 60)

# 「人口」のみに含まれる統計名
unique_population = stats_population - stats_gdp
print(f"\n「人口」のみ: {len(unique_population)}種類")
for stat in sorted(list(unique_population)[:10]):  # 最初の10件
    print(f"  - {stat}")

# 「GDP」のみに含まれる統計名
unique_gdp = stats_gdp - stats_population
print(f"\n「GDP」のみ: {len(unique_gdp)}種類")
for stat in sorted(list(unique_gdp)[:10]):  # 最初の10件
    print(f"  - {stat}")

# ===== ステップ5: ベン図風の可視化 =====
print("\n【ステップ5】ベン図風の表示")
print("=" * 60)

print(f"""
┌─────────────────────────────────────┐
│          統計名の分布               │
├─────────────────────────────────────┤
│ 「人口」のみ      : {len(unique_population):3}種類     │
│ 「GDP」のみ       : {len(unique_gdp):3}種類     │
│ 共通              : {len(common_stats):3}種類     │
├─────────────────────────────────────┤
│ 合計              : {len(stats_population | stats_gdp):3}種類     │
└─────────────────────────────────────┘
""")

# ===== ステップ6: 詳細な比較レポート =====
print("\n【ステップ6】詳細比較レポート")
print("=" * 60)

# 共通統計名のデータを抽出
if len(common_stats) > 0:
    for stat_name in sorted(common_stats):
        print(f"\n■ {stat_name}")

        # 「人口」での該当件数
        pop_count = len(df_population[df_population['STAT_NAME'] == stat_name])
        print(f"  「人口」での登場: {pop_count}件")

        # 「GDP」での該当件数
        gdp_count = len(df_gdp[df_gdp['STAT_NAME'] == stat_name])
        print(f"  「GDP」での登場: {gdp_count}件")
```

---

## 🎯 まとめ

この章では以下を学びました：

✅ StatsListReaderクラスの詳細な使い方
✅ 高度な検索パラメータの活用
✅ 検索結果のフィルタリングとソート
✅ ページネーション機能
✅ 検索結果の保存と再利用
✅ 統計表カタログの作成
✅ 検索結果の比較分析

---

## 📚 次のステップ

次の章では、メタ情報の取得について詳しく学びます：

**[第3章: 中級編 - MetaInfoReaderでメタ情報を取得](./chapter03_metainfo.md)**

- MetaInfoReaderクラスの詳細
- データ構造の理解
- 分類階層の取得と活用
- メタデータを使った効率的なデータ取得

---

お疲れ様でした！
