# 第4章: 中級編 - StatsDataReaderでデータを取得

## 📚 この章で学ぶこと

- StatsDataReaderクラスの詳細な使い方
- 大量データの効率的な取得（自動ページネーション）
- 様々なフィルタリングオプション
- データの前処理とクリーニング
- 効率的なデータ取得戦略

**対象レベル**: 中級者
**所要時間**: 約60分
**前提知識**: 第1〜3章の内容

---

## 4.1 StatsDataReaderとは

### 4.1.1 概要

`StatsDataReader`は、e-Statから実際の統計データを取得するためのクラスです。第3章で学んだメタ情報を活用して、効率的にデータを取得できます。

### 4.1.2 主な機能

- ✅ **自動ページネーション**: 10万件を超えるデータも自動で分割取得
- ✅ **柔軟なフィルタリング**: 地域、時間、カテゴリなどで絞り込み
- ✅ **効率的な取得**: 必要なデータだけを取得
- ✅ **データ変換**: 自動的にpandas DataFrameに変換

---

## 4.2 基本的な使い方

### 4.2.1 シンプルなデータ取得

```python
"""
StatsDataReaderの基本的な使い方
"""
import os
from dotenv import load_dotenv
from jpy_datareader.estat import StatsDataReader
import pandas as pd

# 環境変数を読み込む
load_dotenv()

# StatsDataReaderのインスタンスを作成
reader = StatsDataReader()

# 統計表IDを指定してデータを取得
stats_id = "0003410379"  # 例: 家計調査

# データを取得
df = reader.read(
    statsDataId=stats_id,
    limit=100  # 最初の100件を取得
)

# 結果を確認
print(f"取得データ: {len(df)}行 × {len(df.columns)}列")
print("\n【データの先頭】")
print(df.head())

# カラム一覧を表示
print("\n【カラム一覧】")
print(df.columns.tolist())
```

### 4.2.2 利用可能なパラメータ

```python
"""
StatsDataReaderの全パラメータ
"""
from jpy_datareader.estat import StatsDataReader

reader = StatsDataReader()

df = reader.read(
    # 必須パラメータ
    statsDataId="0003410379",    # 統計表ID

    # データ取得設定
    limit=1000,                   # 取得件数（最大100,000）
    startPosition=1,              # 取得開始位置

    # フィルタリングパラメータ
    cdArea=None,                  # 地域コード
    cdCat01=None,                 # カテゴリ1のコード
    cdCat02=None,                 # カテゴリ2のコード
    cdCat03=None,                 # カテゴリ3のコード
    cdTime=None,                  # 時間コード

    # その他
    lang="J"                      # 言語 (J: 日本語, E: 英語)
)
```

---

## 4.3 フィルタリング機能

### 4.3.1 地域でフィルタリング

```python
"""
地域を指定してデータを取得
"""
from jpy_datareader.estat import MetaInfoReader, StatsDataReader
from dotenv import load_dotenv

load_dotenv()

def get_data_by_area(stats_id, area_name, limit=1000):
    """
    地域名を指定してデータを取得

    Parameters:
    -----------
    stats_id : str
        統計表ID
    area_name : str
        地域名（例: '東京都'）
    limit : int
        取得件数

    Returns:
    --------
    pandas.DataFrame
        取得したデータ
    """
    # ステップ1: メタ情報から地域コードを取得
    meta_reader = MetaInfoReader()
    df_meta = meta_reader.read(statsDataId=stats_id)

    # 地域分類を抽出
    df_area = df_meta[df_meta['@class'] == 'area']

    # 地域名で検索
    df_found = df_area[df_area['@name'].str.contains(area_name, na=False)]

    if len(df_found) == 0:
        print(f"❌ '{area_name}'が見つかりませんでした")
        return None

    # 最初の該当地域のコードを使用
    area_code = df_found.iloc[0]['@code']
    area_full_name = df_found.iloc[0]['@name']

    print(f"✅ 地域: {area_full_name} [{area_code}]")

    # ステップ2: データを取得
    data_reader = StatsDataReader()
    df = data_reader.read(
        statsDataId=stats_id,
        cdArea=area_code,  # 地域コードを指定
        limit=limit
    )

    print(f"✅ {len(df)}件のデータを取得しました")

    return df


# ===== 使用例 =====
print("【地域別データ取得】")
print("=" * 60)

stats_id = "0003410379"

# 東京都のデータを取得
df_tokyo = get_data_by_area(stats_id, "東京都", limit=500)

if df_tokyo is not None:
    print("\n【データの先頭】")
    print(df_tokyo.head())

    # 基本統計量
    numeric_cols = df_tokyo.select_dtypes(include=['float64', 'int64']).columns
    if len(numeric_cols) > 0:
        print("\n【基本統計量】")
        print(df_tokyo[numeric_cols].describe())
```

### 4.3.2 時間でフィルタリング

```python
"""
時間を指定してデータを取得
"""
from jpy_datareader.estat import MetaInfoReader, StatsDataReader
from dotenv import load_dotenv

load_dotenv()

def get_data_by_time(stats_id, time_keyword, limit=1000):
    """
    時間キーワードを指定してデータを取得

    Parameters:
    -----------
    stats_id : str
        統計表ID
    time_keyword : str
        時間キーワード（例: '2024'）
    limit : int
        取得件数

    Returns:
    --------
    pandas.DataFrame
        取得したデータ
    """
    # メタ情報から時間コードを取得
    meta_reader = MetaInfoReader()
    df_meta = meta_reader.read(statsDataId=stats_id)

    # 時間分類を抽出
    df_time = df_meta[df_meta['@class'] == 'time']

    # 時間で検索
    df_found = df_time[df_time['@name'].str.contains(time_keyword, na=False)]

    if len(df_found) == 0:
        print(f"❌ '{time_keyword}'が見つかりませんでした")
        return None

    print(f"✅ {len(df_found)}件の時間が見つかりました")

    # 最初の該当時間のコードを使用
    time_code = df_found.iloc[0]['@code']
    time_name = df_found.iloc[0]['@name']

    print(f"   使用する時間: {time_name} [{time_code}]")

    # データを取得
    data_reader = StatsDataReader()
    df = data_reader.read(
        statsDataId=stats_id,
        cdTime=time_code,  # 時間コードを指定
        limit=limit
    )

    print(f"✅ {len(df)}件のデータを取得しました")

    return df


# 使用例
print("【時間別データ取得】")
print("=" * 60)

stats_id = "0003410379"

# 2024年のデータを取得
df_2024 = get_data_by_time(stats_id, "2024", limit=500)

if df_2024 is not None:
    print("\n【データの先頭】")
    print(df_2024.head(10))
```

### 4.3.3 複数条件での絞り込み

```python
"""
複数の条件を組み合わせてデータを取得
"""
from jpy_datareader.estat import MetaInfoReader, StatsDataReader
from dotenv import load_dotenv

load_dotenv()

def get_filtered_data(
    stats_id,
    area_keyword=None,
    time_keyword=None,
    cat01_keyword=None,
    limit=1000
):
    """
    複数条件でデータを絞り込んで取得

    Parameters:
    -----------
    stats_id : str
        統計表ID
    area_keyword : str, optional
        地域キーワード
    time_keyword : str, optional
        時間キーワード
    cat01_keyword : str, optional
        カテゴリ1のキーワード
    limit : int
        取得件数

    Returns:
    --------
    pandas.DataFrame
        取得したデータ
    """
    # メタ情報を取得
    meta_reader = MetaInfoReader()
    df_meta = meta_reader.read(statsDataId=stats_id)

    # データ取得用のパラメータを構築
    params = {
        'statsDataId': stats_id,
        'limit': limit
    }

    # 地域コードを検索
    if area_keyword:
        df_area = df_meta[df_meta['@class'] == 'area']
        df_found = df_area[df_area['@name'].str.contains(area_keyword, na=False)]

        if len(df_found) > 0:
            params['cdArea'] = df_found.iloc[0]['@code']
            print(f"✅ 地域: {df_found.iloc[0]['@name']} [{params['cdArea']}]")

    # 時間コードを検索
    if time_keyword:
        df_time = df_meta[df_meta['@class'] == 'time']
        df_found = df_time[df_time['@name'].str.contains(time_keyword, na=False)]

        if len(df_found) > 0:
            params['cdTime'] = df_found.iloc[0]['@code']
            print(f"✅ 時間: {df_found.iloc[0]['@name']} [{params['cdTime']}]")

    # カテゴリ1のコードを検索
    if cat01_keyword:
        df_cat = df_meta[df_meta['@class'] == 'cat01']
        df_found = df_cat[df_cat['@name'].str.contains(cat01_keyword, na=False)]

        if len(df_found) > 0:
            params['cdCat01'] = df_found.iloc[0]['@code']
            print(f"✅ カテゴリ: {df_found.iloc[0]['@name']} [{params['cdCat01']}]")

    # データを取得
    data_reader = StatsDataReader()
    df = data_reader.read(**params)

    print(f"\n✅ {len(df)}件のデータを取得しました")

    return df


# ===== 使用例 =====
print("【複数条件でのデータ取得】")
print("=" * 60)

stats_id = "0003410379"

# 大阪府の2024年データを取得
df = get_filtered_data(
    stats_id,
    area_keyword="大阪府",
    time_keyword="2024",
    limit=500
)

if df is not None and len(df) > 0:
    print("\n【データの先頭】")
    print(df.head())

    # データをCSVに保存
    output_file = "osaka_2024_data.csv"
    df.to_csv(output_file, index=False, encoding='utf-8-sig')
    print(f"\n✅ データを {output_file} に保存しました")
```

---

## 4.4 大量データの取得（自動ページネーション）

### 4.4.1 自動ページネーションの仕組み

```python
"""
大量データの自動取得
"""
from jpy_datareader.estat import StatsDataReader
from dotenv import load_dotenv
import time

load_dotenv()

def get_large_dataset(stats_id, max_records=None):
    """
    大量データを自動ページネーションで取得

    Parameters:
    -----------
    stats_id : str
        統計表ID
    max_records : int, optional
        最大取得件数（Noneの場合はすべて取得）

    Returns:
    --------
    pandas.DataFrame
        取得したデータ
    """
    reader = StatsDataReader()

    # limitを指定しない、またはNoneにすると自動ページネーション
    start_time = time.time()

    print(f"【大量データ取得開始】")
    print("=" * 60)
    print(f"統計表ID: {stats_id}")

    if max_records:
        df = reader.read(statsDataId=stats_id, limit=max_records)
    else:
        # limitを指定しないと全データ取得（自動ページネーション）
        df = reader.read(statsDataId=stats_id)

    elapsed_time = time.time() - start_time

    print(f"\n✅ 取得完了")
    print(f"   データ件数: {len(df):,}件")
    print(f"   カラム数: {len(df.columns)}列")
    print(f"   所要時間: {elapsed_time:.2f}秒")

    return df


# 使用例
stats_id = "0003410379"

# 10,000件取得
df = get_large_dataset(stats_id, max_records=10000)

if df is not None:
    print("\n【データ概要】")
    print(df.info())
```

### 4.4.2 手動ページネーション

```python
"""
手動でページネーションを制御
"""
from jpy_datareader.estat import StatsDataReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

def manual_pagination(stats_id, page_size=10000, max_pages=5):
    """
    手動でページネーションを制御してデータ取得

    Parameters:
    -----------
    stats_id : str
        統計表ID
    page_size : int
        1ページあたりの件数
    max_pages : int
        最大ページ数

    Returns:
    --------
    pandas.DataFrame
        結合したデータ
    """
    reader = StatsDataReader()
    all_data = []

    print(f"【手動ページネーション】")
    print("=" * 60)

    for page in range(max_pages):
        start_pos = page * page_size + 1

        print(f"\nページ {page + 1}/{max_pages} を取得中...")
        print(f"  開始位置: {start_pos}")

        try:
            df_page = reader.read(
                statsDataId=stats_id,
                startPosition=start_pos,
                limit=page_size
            )

            if len(df_page) == 0:
                print(f"  データがありません（取得終了）")
                break

            all_data.append(df_page)
            print(f"  ✅ {len(df_page)}件取得")

            # 取得件数がpage_sizeより少なければ終了
            if len(df_page) < page_size:
                print(f"  最終ページに到達")
                break

        except Exception as e:
            print(f"  ❌ エラー: {e}")
            break

    # すべてのデータを結合
    if all_data:
        df_combined = pd.concat(all_data, ignore_index=True)
        print(f"\n✅ 総取得件数: {len(df_combined):,}件")
        return df_combined
    else:
        print(f"\n❌ データが取得できませんでした")
        return None


# 使用例
stats_id = "0003410379"

df = manual_pagination(stats_id, page_size=5000, max_pages=3)

if df is not None:
    print("\n【データの先頭と末尾】")
    print("\n先頭5行:")
    print(df.head())
    print("\n末尾5行:")
    print(df.tail())
```

---

## 4.5 データの前処理とクリーニング

### 4.5.1 基本的な前処理

```python
"""
取得したデータの基本的な前処理
"""
from jpy_datareader.estat import StatsDataReader
import pandas as pd
import numpy as np
from dotenv import load_dotenv

load_dotenv()

def preprocess_data(df):
    """
    データの前処理

    Parameters:
    -----------
    df : pandas.DataFrame
        元のデータ

    Returns:
    --------
    pandas.DataFrame
        前処理後のデータ
    """
    print("【データ前処理】")
    print("=" * 60)

    # コピーを作成
    df_clean = df.copy()

    # ===== ステップ1: 欠損値の確認 =====
    print("\nステップ1: 欠損値の確認")
    print("-" * 60)

    missing_counts = df_clean.isnull().sum()
    missing_cols = missing_counts[missing_counts > 0]

    if len(missing_cols) > 0:
        print("欠損値があるカラム:")
        for col, count in missing_cols.items():
            percentage = (count / len(df_clean)) * 100
            print(f"  {col}: {count}件 ({percentage:.1f}%)")
    else:
        print("✅ 欠損値はありません")

    # ===== ステップ2: データ型の確認と変換 =====
    print("\nステップ2: データ型の確認")
    print("-" * 60)

    # 数値っぽいカラムを自動検出
    for col in df_clean.columns:
        # 文字列型のカラムで数値に変換できそうなもの
        if df_clean[col].dtype == 'object':
            # 数値に変換を試みる
            try:
                df_clean[col] = pd.to_numeric(df_clean[col], errors='coerce')
                converted = df_clean[col].notna().sum()
                if converted > 0:
                    print(f"  {col}: 数値型に変換 ({converted}件)")
            except:
                pass

    # ===== ステップ3: 重複の確認 =====
    print("\nステップ3: 重複の確認")
    print("-" * 60)

    duplicates = df_clean.duplicated().sum()
    if duplicates > 0:
        print(f"⚠️ 重複行: {duplicates}件")
        print("  重複を削除しますか？")
        # ここでは自動的に削除
        df_clean = df_clean.drop_duplicates()
        print(f"  ✅ 重複を削除しました（残り: {len(df_clean)}件）")
    else:
        print("✅ 重複はありません")

    # ===== ステップ4: 基本統計量 =====
    print("\nステップ4: 基本統計量")
    print("-" * 60)

    numeric_cols = df_clean.select_dtypes(include=[np.number]).columns
    if len(numeric_cols) > 0:
        print(f"数値カラム数: {len(numeric_cols)}")
        print("\n基本統計量:")
        print(df_clean[numeric_cols].describe())

    print(f"\n✅ 前処理完了（{len(df_clean)}件）")

    return df_clean


# ===== 使用例 =====
reader = StatsDataReader()
stats_id = "0003410379"

# データを取得
df_raw = reader.read(statsDataId=stats_id, limit=1000)

print(f"【元データ】{len(df_raw)}件\n")

# 前処理
df_processed = preprocess_data(df_raw)

# 結果を保存
df_processed.to_csv("processed_data.csv", index=False, encoding='utf-8-sig')
print("\n✅ 処理済みデータを保存しました")
```

---

## 📝 練習問題

### 問題1: 基本的なデータ取得

**課題**: StatsDataReaderを使って統計表ID "0003410379" から1000件のデータを取得し、データの形状（行数×列数）とカラム名を表示してください。

<details>
<summary>ヒント</summary>

- `StatsDataReader()`でインスタンス作成
- `read(statsDataId=..., limit=1000)`
- `df.shape`で形状を確認
- `df.columns.tolist()`でカラム名を取得

</details>

---

### 問題2: 地域フィルタリング

**課題**: 「福岡県」のデータだけを500件取得し、数値カラムの基本統計量を表示してください。

<details>
<summary>ヒント</summary>

- MetaInfoReaderで地域コードを検索
- StatsDataReaderで`cdArea`を指定
- `select_dtypes(include=['float64', 'int64'])`で数値カラムを抽出
- `describe()`で統計量を表示

</details>

---

### 問題3: 複数条件フィルタリング

**課題**: 「北海道」の「2023年」のデータを取得し、CSVファイルに保存してください。

<details>
<summary>ヒント</summary>

- 地域と時間の両方でフィルタ
- `cdArea`と`cdTime`を両方指定
- `to_csv()`で保存

</details>

---

### 問題4: ページネーション

**課題**: 手動ページネーションを使って、10,000件ずつ3ページ分（合計30,000件）のデータを取得し、結合してください。

<details>
<summary>ヒント</summary>

- `for`ループで3回取得
- `startPosition`を1, 10001, 20001と変更
- `pd.concat()`で結合

</details>

---

### 問題5: データクリーニング

**課題**: データを取得した後、以下の処理を行ってください：
1. 欠損値を含む行を削除
2. 重複行を削除
3. 数値カラムの平均値を計算
4. 処理済みデータをCSVに保存

<details>
<summary>ヒント</summary>

- `dropna()`で欠損値削除
- `drop_duplicates()`で重複削除
- `mean()`で平均値計算
- `to_csv()`で保存

</details>

---

## 📖 模範解答

### 解答1: 基本的なデータ取得

```python
"""
問題1の模範解答: 基本的なデータ取得
"""
from jpy_datareader.estat import StatsDataReader
from dotenv import load_dotenv

load_dotenv()

# StatsDataReaderのインスタンスを作成
reader = StatsDataReader()

# 統計表IDを指定
stats_id = "0003410379"

print("【データ取得】")
print("=" * 60)
print(f"統計表ID: {stats_id}")

# データを取得
df = reader.read(statsDataId=stats_id, limit=1000)

# データの形状を表示
rows, cols = df.shape
print(f"\n✅ データ取得完了")
print(f"   形状: {rows}行 × {cols}列")

# カラム名を表示
print(f"\n【カラム一覧】({len(df.columns)}個)")
print("-" * 60)

for i, col in enumerate(df.columns, 1):
    print(f"{i:2}. {col}")

# データ型も表示
print(f"\n【データ型】")
print("-" * 60)
print(df.dtypes)

# メモリ使用量
memory_mb = df.memory_usage(deep=True).sum() / 1024 / 1024
print(f"\n【メモリ使用量】")
print(f"約 {memory_mb:.2f} MB")
```

---

### 解答2: 地域フィルタリング

```python
"""
問題2の模範解答: 福岡県データの取得と分析
"""
from jpy_datareader.estat import MetaInfoReader, StatsDataReader
from dotenv import load_dotenv

load_dotenv()

print("【福岡県データの取得】")
print("=" * 60)

stats_id = "0003410379"

# ステップ1: メタ情報から福岡県のコードを検索
print("\nステップ1: 福岡県のコードを検索")
print("-" * 60)

meta_reader = MetaInfoReader()
df_meta = meta_reader.read(statsDataId=stats_id)

# 地域分類を抽出
df_area = df_meta[df_meta['@class'] == 'area']

# 福岡県を検索
df_fukuoka = df_area[df_area['@name'].str.contains('福岡県', na=False)]

if len(df_fukuoka) == 0:
    print("❌ 福岡県が見つかりませんでした")
else:
    fukuoka_code = df_fukuoka.iloc[0]['@code']
    fukuoka_name = df_fukuoka.iloc[0]['@name']

    print(f"✅ 福岡県コード: [{fukuoka_code}] {fukuoka_name}")

    # ステップ2: データを取得
    print("\nステップ2: データを取得")
    print("-" * 60)

    data_reader = StatsDataReader()
    df = data_reader.read(
        statsDataId=stats_id,
        cdArea=fukuoka_code,
        limit=500
    )

    print(f"✅ {len(df)}件のデータを取得")

    # ステップ3: 数値カラムの基本統計量
    print("\nステップ3: 基本統計量の計算")
    print("-" * 60)

    numeric_cols = df.select_dtypes(include=['float64', 'int64']).columns

    if len(numeric_cols) > 0:
        print(f"数値カラム数: {len(numeric_cols)}\n")

        # すべての数値カラムの基本統計量
        print("【基本統計量】")
        print(df[numeric_cols].describe())

        # 個別に詳細を表示
        print("\n【各カラムの詳細】")
        for col in numeric_cols:
            print(f"\n■ {col}")
            print(f"  平均値: {df[col].mean():.2f}")
            print(f"  中央値: {df[col].median():.2f}")
            print(f"  標準偏差: {df[col].std():.2f}")
            print(f"  最小値: {df[col].min():.2f}")
            print(f"  最大値: {df[col].max():.2f}")
            print(f"  欠損値: {df[col].isnull().sum()}件")
    else:
        print("⚠️ 数値カラムが見つかりませんでした")
```

---

### 解答3: 複数条件フィルタリング

```python
"""
問題3の模範解答: 北海道の2023年データ取得
"""
from jpy_datareader.estat import MetaInfoReader, StatsDataReader
from dotenv import load_dotenv
import os

load_dotenv()

print("【北海道の2023年データ取得】")
print("=" * 60)

stats_id = "0003410379"

# メタ情報を取得
meta_reader = MetaInfoReader()
df_meta = meta_reader.read(statsDataId=stats_id)

# パラメータを準備
params = {
    'statsDataId': stats_id,
    'limit': 1000
}

# ステップ1: 北海道のコードを検索
print("\nステップ1: 北海道のコードを検索")
print("-" * 60)

df_area = df_meta[df_meta['@class'] == 'area']
df_hokkaido = df_area[df_area['@name'].str.contains('北海道', na=False)]

if len(df_hokkaido) > 0:
    params['cdArea'] = df_hokkaido.iloc[0]['@code']
    print(f"✅ 地域: {df_hokkaido.iloc[0]['@name']} [{params['cdArea']}]")
else:
    print("❌ 北海道が見つかりませんでした")

# ステップ2: 2023年のコードを検索
print("\nステップ2: 2023年のコードを検索")
print("-" * 60)

df_time = df_meta[df_meta['@class'] == 'time']
df_2023 = df_time[df_time['@name'].str.contains('2023', na=False)]

if len(df_2023) > 0:
    params['cdTime'] = df_2023.iloc[0]['@code']
    print(f"✅ 時間: {df_2023.iloc[0]['@name']} [{params['cdTime']}]")

    # 2023年のデータがいくつあるか表示
    print(f"   2023年の時点数: {len(df_2023)}件")

    if len(df_2023) > 1:
        print("\n   2023年の時点一覧:")
        for idx, row in df_2023.head(5).iterrows():
            print(f"     - [{row['@code']}] {row['@name']}")
else:
    print("⚠️ 2023年のデータが見つかりませんでした")

# ステップ3: データを取得
print("\nステップ3: データを取得")
print("-" * 60)

if 'cdArea' in params and 'cdTime' in params:
    data_reader = StatsDataReader()
    df = data_reader.read(**params)

    print(f"✅ {len(df)}件のデータを取得")

    # ステップ4: CSVに保存
    print("\nステップ4: CSVに保存")
    print("-" * 60)

    output_file = "hokkaido_2023_data.csv"
    df.to_csv(output_file, index=False, encoding='utf-8-sig')

    file_size = os.path.getsize(output_file)
    print(f"✅ データを {output_file} に保存しました")
    print(f"   ファイルサイズ: {file_size:,} bytes ({file_size/1024:.1f} KB)")

    # データの概要を表示
    print("\n【データ概要】")
    print(df.head(10))

else:
    print("\n❌ 必要な条件が揃いませんでした")
```

---

### 解答4: ページネーション

```python
"""
問題4の模範解答: 手動ページネーション
"""
from jpy_datareader.estat import StatsDataReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

print("【手動ページネーションでデータ取得】")
print("=" * 60)

stats_id = "0003410379"
page_size = 10000
num_pages = 3

reader = StatsDataReader()
all_pages = []

# 各ページを取得
for page_num in range(num_pages):
    start_pos = page_num * page_size + 1

    print(f"\n【ページ {page_num + 1}/{num_pages}】")
    print(f"開始位置: {start_pos:,}")

    try:
        df_page = reader.read(
            statsDataId=stats_id,
            startPosition=start_pos,
            limit=page_size
        )

        print(f"✅ {len(df_page):,}件取得")

        # ページをリストに追加
        all_pages.append(df_page)

        # データがpage_sizeより少なければ終了
        if len(df_page) < page_size:
            print(f"⚠️ 最終ページに到達（{len(df_page):,}件のみ取得）")
            break

    except Exception as e:
        print(f"❌ エラー発生: {e}")
        break

# すべてのページを結合
print("\n【データの結合】")
print("=" * 60)

if all_pages:
    df_combined = pd.concat(all_pages, ignore_index=True)

    print(f"✅ 結合完了")
    print(f"   総データ件数: {len(df_combined):,}件")
    print(f"   カラム数: {len(df_combined.columns)}列")
    print(f"   ページ数: {len(all_pages)}ページ")

    # 各ページの件数を表示
    print("\n【ページ別件数】")
    for i, df_page in enumerate(all_pages, 1):
        print(f"   ページ{i}: {len(df_page):,}件")

    # データの先頭と末尾を表示
    print("\n【データの先頭5行】")
    print(df_combined.head())

    print("\n【データの末尾5行】")
    print(df_combined.tail())

    # 重複チェック
    duplicates = df_combined.duplicated().sum()
    print(f"\n【重複チェック】")
    print(f"   重複行数: {duplicates:,}件")

    # 保存
    output_file = "paginated_data.csv"
    df_combined.to_csv(output_file, index=False, encoding='utf-8-sig')
    print(f"\n✅ データを {output_file} に保存しました")

else:
    print("❌ データが取得できませんでした")
```

---

### 解答5: データクリーニング

```python
"""
問題5の模範解答: データクリーニング
"""
from jpy_datareader.estat import StatsDataReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

print("【データクリーニング】")
print("=" * 60)

stats_id = "0003410379"

# データを取得
reader = StatsDataReader()
df_raw = reader.read(statsDataId=stats_id, limit=5000)

print(f"\n【元データ】")
print(f"データ件数: {len(df_raw):,}件")
print(f"カラム数: {len(df_raw.columns)}列")

# ===== ステップ1: 欠損値を含む行を削除 =====
print("\n【ステップ1】欠損値の処理")
print("-" * 60)

# 欠損値の確認
missing_before = df_raw.isnull().sum().sum()
print(f"欠損値の総数: {missing_before:,}個")

# 欠損値を含む行を削除
df_clean = df_raw.dropna()

removed_by_na = len(df_raw) - len(df_clean)
print(f"✅ 欠損値を含む行を削除: {removed_by_na:,}行")
print(f"   残り: {len(df_clean):,}行")

# ===== ステップ2: 重複行を削除 =====
print("\n【ステップ2】重複行の処理")
print("-" * 60)

duplicates_count = df_clean.duplicated().sum()
print(f"重複行数: {duplicates_count:,}行")

df_clean = df_clean.drop_duplicates()

removed_by_dup = duplicates_count
print(f"✅ 重複行を削除: {removed_by_dup:,}行")
print(f"   残り: {len(df_clean):,}行")

# ===== ステップ3: 数値カラムの平均値を計算 =====
print("\n【ステップ3】数値カラムの平均値")
print("-" * 60)

numeric_cols = df_clean.select_dtypes(include=['float64', 'int64']).columns

if len(numeric_cols) > 0:
    print(f"数値カラム数: {len(numeric_cols)}\n")

    # 各カラムの平均値を計算
    for col in numeric_cols:
        mean_value = df_clean[col].mean()
        median_value = df_clean[col].median()

        print(f"■ {col}")
        print(f"  平均値: {mean_value:,.2f}")
        print(f"  中央値: {median_value:,.2f}")
        print()

    # 平均値のサマリーDataFrameを作成
    summary = pd.DataFrame({
        'カラム': numeric_cols,
        '平均値': [df_clean[col].mean() for col in numeric_cols],
        '中央値': [df_clean[col].median() for col in numeric_cols],
        '標準偏差': [df_clean[col].std() for col in numeric_cols],
        '最小値': [df_clean[col].min() for col in numeric_cols],
        '最大値': [df_clean[col].max() for col in numeric_cols]
    })

    print("【統計サマリー】")
    print(summary.to_string(index=False))

else:
    print("⚠️ 数値カラムが見つかりませんでした")

# ===== ステップ4: 処理済みデータをCSVに保存 =====
print("\n【ステップ4】データの保存")
print("-" * 60)

output_file = "cleaned_data.csv"
df_clean.to_csv(output_file, index=False, encoding='utf-8-sig')

print(f"✅ クリーニング済みデータを {output_file} に保存")
print(f"   最終データ件数: {len(df_clean):,}行")
print(f"   削除された行数: {len(df_raw) - len(df_clean):,}行")
print(f"   保持率: {(len(df_clean) / len(df_raw) * 100):.1f}%")

# サマリーも保存
if len(numeric_cols) > 0:
    summary_file = "data_summary.csv"
    summary.to_csv(summary_file, index=False, encoding='utf-8-sig')
    print(f"✅ 統計サマリーを {summary_file} に保存")

# クリーニング結果のレポート
print("\n【クリーニングレポート】")
print("=" * 60)
print(f"元データ件数: {len(df_raw):,}行")
print(f"欠損値削除: -{removed_by_na:,}行")
print(f"重複削除: -{removed_by_dup:,}行")
print(f"最終データ: {len(df_clean):,}行")
print(f"削除率: {((len(df_raw) - len(df_clean)) / len(df_raw) * 100):.1f}%")
```

---

## 🎯 まとめ

この章では以下を学びました：

✅ StatsDataReaderクラスの詳細な使い方
✅ 様々なフィルタリングオプション（地域、時間、カテゴリ）
✅ 大量データの自動ページネーション機能
✅ 手動ページネーションの制御方法
✅ データの前処理とクリーニング技術
✅ 効率的なデータ取得戦略

---

## 📚 次のステップ

次の章では、実践的なデータ分析を学びます：

**[第5章: 上級編 - データ分析の実践](./chapter05_advanced_analysis.md)**

- 複数統計表の結合
- 時系列分析
- 地域比較分析
- データ可視化
- 実践プロジェクト

---

お疲れ様でした！
