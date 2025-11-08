# 第1章: 基礎編 - はじめてのJPy-DataReader

## 📚 この章で学ぶこと

- JPy-DataReaderのインストール方法
- e-Stat APIキーの取得と設定
- 基本的な3つの関数の使い方
- 最初の統計データ取得
- データの基本的な操作

**対象レベル**: 初心者
**所要時間**: 約30分
**前提知識**: Pythonの基本文法、pandasの基礎

---

## 1.1 環境構築

### 1.1.1 必要な環境

JPy-DataReaderを使用するには以下が必要です：

- **Python 3.12以上**
- **pip**（Pythonパッケージマネージャー）
- インターネット接続

### 1.1.2 インストール

ターミナルまたはコマンドプロンプトで以下を実行：

```bash
# JPy-DataReaderのインストール
pip install jpy-datareader

# 依存パッケージも自動的にインストールされます
# - pandas
# - numpy
# - requests
# - python-dotenv
```

### 1.1.3 インストールの確認

```python
# インストールが成功したか確認
import jpy_datareader
print(jpy_datareader.__version__)
```

---

## 1.2 e-Stat APIキーの取得

### 1.2.1 APIキーとは？

e-Stat APIを利用するには、**アプリケーションID**（APIキー）が必要です。これは無料で取得できます。

### 1.2.2 取得手順

1. **e-Stat API 利用登録ページにアクセス**
   - URL: https://www.e-stat.go.jp/api/

2. **利用規約を確認**
   - 統計データの利用目的を確認
   - 個人情報保護方針を確認

3. **ユーザー登録**
   - メールアドレスを入力
   - 確認メールが届く
   - 指示に従って登録完了

4. **アプリケーションIDを取得**
   - ログイン後、アプリケーションIDを発行
   - このIDを控えておく（後で使用）

### 1.2.3 環境変数の設定

APIキーを安全に管理するため、`.env`ファイルを使用します。

**プロジェクトディレクトリに `.env` ファイルを作成：**

```bash
# .env ファイルの内容
ESTAT_APP_ID=あなたのアプリケーションID
```

**重要**: `.env`ファイルは`.gitignore`に追加し、GitHubなどにアップロードしないようにしてください。

---

## 1.3 基本的な3つの関数

JPy-DataReaderには、簡単に使える3つの便利関数があります：

| 関数名 | 用途 | 戻り値 |
|--------|------|--------|
| `get_data_estat_statslist()` | 統計表を検索 | pandas DataFrame |
| `get_data_estat_metainfo()` | メタ情報を取得 | pandas DataFrame |
| `get_data_estat_statsdata()` | 統計データを取得 | pandas DataFrame |

---

## 1.4 統計表の検索（StatsListの取得）

### 1.4.1 基本的な使い方

```python
# 必要なライブラリをインポート
import os
from dotenv import load_dotenv
from jpy_datareader.data import get_data_estat_statslist

# .envファイルから環境変数を読み込む
load_dotenv()

# 環境変数からAPIキーを取得（自動的に使用される）
# os.environ['ESTAT_APP_ID'] が自動的に参照されます

# 「人口」というキーワードで統計表を検索
# searchWord: 検索キーワード
# limit: 取得する件数（最大100,000件）
df_list = get_data_estat_statslist(
    searchWord="人口",  # 検索キーワード
    limit=10            # 最大10件取得
)

# 結果を表示
print(df_list.head())
```

### 1.4.2 結果の確認

```python
# 取得したデータの形状を確認
print(f"取得した統計表の数: {len(df_list)}")

# 主要なカラムだけを表示
print(df_list[['@id', 'STAT_NAME', 'TITLE']].head())

# 出力例:
#           @id                    STAT_NAME                              TITLE
# 0  0000010101                    国勢調査              平成27年国勢調査 人口等基本集計
# 1  0000010102                    国勢調査                  平成27年国勢調査 年齢・国籍
# 2  0000010201              人口推計月報                        人口推計（2020年）
```

### 1.4.3 詳しいコメント付きサンプルコード

```python
"""
統計表検索の詳細サンプル
"""
import os
from dotenv import load_dotenv
from jpy_datareader.data import get_data_estat_statslist
import pandas as pd

# .envファイルから環境変数を読み込む
# プロジェクトルートに.envファイルが必要
load_dotenv()

# 統計表を検索する関数
def search_statistics(keyword, max_results=10):
    """
    指定したキーワードで統計表を検索する

    Parameters:
    -----------
    keyword : str
        検索キーワード（例: 「人口」「GDP」「賃金」）
    max_results : int
        取得する最大件数（デフォルト: 10）

    Returns:
    --------
    pandas.DataFrame
        検索結果のDataFrame
    """
    try:
        # get_data_estat_statslist()を実行
        # この関数は自動的に環境変数ESTAT_APP_IDを参照します
        df = get_data_estat_statslist(
            searchWord=keyword,   # 検索キーワード
            limit=max_results     # 取得件数の上限
        )

        # 取得成功メッセージ
        print(f"✅ '{keyword}'で{len(df)}件の統計表が見つかりました")

        return df

    except Exception as e:
        # エラーが発生した場合
        print(f"❌ エラーが発生しました: {e}")
        return None

# 実行例1: 人口統計を検索
print("=" * 50)
print("例1: 人口統計の検索")
print("=" * 50)
df_population = search_statistics("人口", max_results=5)

if df_population is not None:
    # 統計表のID、統計名、タイトルを表示
    print("\n【検索結果】")
    for idx, row in df_population.iterrows():
        print(f"{idx + 1}. ID: {row['@id']}")
        print(f"   統計名: {row['STAT_NAME']}")
        print(f"   タイトル: {row['TITLE'][:50]}...")  # 最初の50文字だけ表示
        print()

# 実行例2: GDP関連の統計を検索
print("=" * 50)
print("例2: GDP統計の検索")
print("=" * 50)
df_gdp = search_statistics("GDP", max_results=3)

if df_gdp is not None:
    # すべてのカラム名を表示
    print("\n【利用可能なカラム一覧】")
    print(df_gdp.columns.tolist())
```

---

## 1.5 メタ情報の取得

### 1.5.1 基本的な使い方

メタ情報とは、統計データの「構造」に関する情報です。どのような分類項目があるか、どの地域が含まれているかなどがわかります。

```python
from jpy_datareader.data import get_data_estat_metainfo

# 統計表ID（前のステップで取得したID）を指定
stats_id = "0000010101"  # 例: 国勢調査のID

# メタ情報を取得
df_meta = get_data_estat_metainfo(statsDataId=stats_id)

# 結果を表示
print(df_meta.head())
```

### 1.5.2 詳しいコメント付きサンプルコード

```python
"""
メタ情報取得の詳細サンプル
"""
from jpy_datareader.data import get_data_estat_metainfo
import pandas as pd

def get_metadata(stats_data_id):
    """
    指定した統計表IDのメタ情報を取得する

    Parameters:
    -----------
    stats_data_id : str
        統計表のID（例: "0000010101"）

    Returns:
    --------
    pandas.DataFrame
        メタ情報のDataFrame
    """
    try:
        # メタ情報を取得
        # この関数も自動的に環境変数ESTAT_APP_IDを参照します
        df = get_data_estat_metainfo(statsDataId=stats_data_id)

        print(f"✅ 統計表ID {stats_data_id} のメタ情報を取得しました")
        print(f"   データ件数: {len(df)}")

        return df

    except Exception as e:
        print(f"❌ エラーが発生しました: {e}")
        return None

# 実行例
print("=" * 50)
print("メタ情報の取得")
print("=" * 50)

# 統計表ID（前のステップで取得したものを使用）
stats_id = "0003410379"  # 例: 家計調査

df_metadata = get_metadata(stats_id)

if df_metadata is not None:
    # メタ情報の内容を確認
    print("\n【メタ情報の構造】")
    print(f"カラム数: {len(df_metadata.columns)}")
    print(f"データ行数: {len(df_metadata)}")

    # 最初の5行を表示
    print("\n【最初の5行】")
    print(df_metadata.head())

    # どのような分類があるか確認
    if '@class' in df_metadata.columns:
        print("\n【分類の種類】")
        print(df_metadata['@class'].unique())
```

---

## 1.6 統計データの取得

### 1.6.1 基本的な使い方

```python
from jpy_datareader.data import get_data_estat_statsdata

# 統計表IDを指定してデータを取得
stats_id = "0000010101"

# データを取得
df_data = get_data_estat_statsdata(statsDataId=stats_id)

# 結果を表示
print(df_data.head())
print(f"\nデータの形状: {df_data.shape}")
```

### 1.6.2 詳しいコメント付きサンプルコード

```python
"""
統計データ取得の詳細サンプル
"""
from jpy_datareader.data import get_data_estat_statsdata
import pandas as pd

def get_statistics_data(stats_data_id, limit=None):
    """
    指定した統計表IDの実データを取得する

    Parameters:
    -----------
    stats_data_id : str
        統計表のID
    limit : int, optional
        取得するデータの最大件数（Noneの場合はすべて取得）

    Returns:
    --------
    pandas.DataFrame
        統計データのDataFrame
    """
    try:
        # 統計データを取得
        # limit=Noneの場合、自動的にページネーションして全データを取得
        if limit:
            df = get_data_estat_statsdata(
                statsDataId=stats_data_id,
                limit=limit
            )
        else:
            df = get_data_estat_statsdata(
                statsDataId=stats_data_id
            )

        print(f"✅ 統計データを取得しました")
        print(f"   データ行数: {len(df)}")
        print(f"   カラム数: {len(df.columns)}")

        return df

    except Exception as e:
        print(f"❌ エラーが発生しました: {e}")
        return None

# 実行例
print("=" * 50)
print("統計データの取得")
print("=" * 50)

# 統計表ID
stats_id = "0003410379"  # 例: 家計調査

# データを取得（最初の1000件のみ）
df_stats = get_statistics_data(stats_id, limit=1000)

if df_stats is not None:
    # データの概要を表示
    print("\n【データの概要】")
    print(df_stats.info())

    # 最初の10行を表示
    print("\n【最初の10行】")
    print(df_stats.head(10))

    # 数値データがあるカラムを確認
    print("\n【数値カラム】")
    numeric_cols = df_stats.select_dtypes(include=['float64', 'int64']).columns
    print(numeric_cols.tolist())

    # 基本統計量を表示
    if len(numeric_cols) > 0:
        print("\n【基本統計量】")
        print(df_stats[numeric_cols].describe())
```

---

## 1.7 実践: 完全なワークフロー

実際の分析フローを体験してみましょう。

```python
"""
完全なデータ取得ワークフロー
ステップ1: 統計表を検索
ステップ2: メタ情報を確認
ステップ3: データを取得
ステップ4: 基本的な分析
"""

import os
from dotenv import load_dotenv
from jpy_datareader.data import (
    get_data_estat_statslist,
    get_data_estat_metainfo,
    get_data_estat_statsdata
)
import pandas as pd

# 環境設定
load_dotenv()

# ===== ステップ1: 統計表を検索 =====
print("【ステップ1】統計表の検索")
print("-" * 50)

# 「家計」に関する統計を検索
df_list = get_data_estat_statslist(searchWord="家計", limit=5)

# 検索結果を表示
print(f"検索結果: {len(df_list)}件")
print("\n統計表一覧:")
for idx, row in df_list.iterrows():
    print(f"{idx + 1}. {row['STAT_NAME']} - {row['TITLE'][:40]}...")

# 最初の統計表のIDを取得
target_stats_id = df_list.iloc[0]['@id']
print(f"\n選択した統計表ID: {target_stats_id}")

# ===== ステップ2: メタ情報を確認 =====
print("\n【ステップ2】メタ情報の取得")
print("-" * 50)

# メタ情報を取得
df_meta = get_data_estat_metainfo(statsDataId=target_stats_id)

print(f"メタ情報: {len(df_meta)}件")

# メタ情報の分類を確認
if '@class' in df_meta.columns:
    print("\n分類一覧:")
    for cls in df_meta['@class'].unique():
        count = len(df_meta[df_meta['@class'] == cls])
        print(f"  - {cls}: {count}件")

# ===== ステップ3: データを取得 =====
print("\n【ステップ3】統計データの取得")
print("-" * 50)

# データを取得（最初の100件）
df_data = get_data_estat_statsdata(
    statsDataId=target_stats_id,
    limit=100
)

print(f"データ行数: {len(df_data)}")
print(f"カラム数: {len(df_data.columns)}")

# ===== ステップ4: 基本的な分析 =====
print("\n【ステップ4】基本的な分析")
print("-" * 50)

# データの先頭を表示
print("\nデータの先頭5行:")
print(df_data.head())

# 数値カラムがあれば統計量を表示
numeric_cols = df_data.select_dtypes(include=['float64', 'int64']).columns
if len(numeric_cols) > 0:
    print("\n基本統計量:")
    print(df_data[numeric_cols].describe())

# データをCSVに保存
output_file = "estat_data_sample.csv"
df_data.to_csv(output_file, index=False, encoding='utf-8-sig')
print(f"\n✅ データを {output_file} に保存しました")
```

---

## 📝 練習問題

### 問題1: 基本的な検索

**課題**: 「賃金」というキーワードで統計表を検索し、最初の3件のタイトルを表示してください。

<details>
<summary>ヒント</summary>

- `get_data_estat_statslist()`を使用
- `searchWord="賃金"`を指定
- `limit=3`で件数を制限
- `df['TITLE']`でタイトルを取得

</details>

---

### 問題2: メタ情報の取得

**課題**: 問題1で取得した最初の統計表のメタ情報を取得し、分類（@class）の種類を表示してください。

<details>
<summary>ヒント</summary>

- 問題1の結果から`@id`を取得
- `get_data_estat_metainfo()`を使用
- `df['@class'].unique()`で分類を取得

</details>

---

### 問題3: データの保存

**課題**: 「GDP」で検索した統計表の最初のデータを100件取得し、CSVファイルに保存してください。

<details>
<summary>ヒント</summary>

- ステップ1: `get_data_estat_statslist(searchWord="GDP")`
- ステップ2: 最初の統計表IDを取得
- ステップ3: `get_data_estat_statsdata(limit=100)`
- ステップ4: `df.to_csv()`で保存

</details>

---

### 問題4: エラーハンドリング

**課題**: 存在しない統計表ID（例: "9999999999"）でデータを取得しようとしたときに、適切なエラーメッセージを表示するコードを書いてください。

<details>
<summary>ヒント</summary>

- `try-except`文を使用
- `Exception`をキャッチ
- エラーメッセージを`print()`で表示

</details>

---

### 問題5: データの基本分析

**課題**: 「人口」で検索した統計データを取得し、数値カラムがあれば平均値を計算して表示してください。

<details>
<summary>ヒント</summary>

- `select_dtypes(include=['float64', 'int64'])`で数値カラムを抽出
- `df.mean()`で平均値を計算
- `for`ループでカラムごとに表示

</details>

---

## 📖 模範解答

### 解答1: 基本的な検索

```python
"""
問題1の模範解答: 賃金統計の検索
"""
from jpy_datareader.data import get_data_estat_statslist
from dotenv import load_dotenv

# 環境変数を読み込む
load_dotenv()

# 「賃金」で検索
df = get_data_estat_statslist(searchWord="賃金", limit=3)

# タイトルを表示
print("【賃金に関する統計表（上位3件）】")
for idx, title in enumerate(df['TITLE'], 1):
    print(f"{idx}. {title}")

# 別の表示方法
print("\n【別の表示方法】")
print(df[['@id', 'STAT_NAME', 'TITLE']])
```

**実行結果例:**
```
【賃金に関する統計表(上位3件)】
1. 毎月勤労統計調査 全国調査 月次
2. 賃金構造基本統計調査 産業別
3. 賃金引上げ等の実態に関する調査
```

---

### 解答2: メタ情報の取得

```python
"""
問題2の模範解答: メタ情報から分類を取得
"""
from jpy_datareader.data import get_data_estat_statslist, get_data_estat_metainfo
from dotenv import load_dotenv

load_dotenv()

# ステップ1: 賃金統計を検索
df_list = get_data_estat_statslist(searchWord="賃金", limit=3)

# ステップ2: 最初の統計表IDを取得
stats_id = df_list.iloc[0]['@id']
print(f"統計表ID: {stats_id}")
print(f"統計名: {df_list.iloc[0]['STAT_NAME']}")

# ステップ3: メタ情報を取得
df_meta = get_data_estat_metainfo(statsDataId=stats_id)

# ステップ4: 分類の種類を表示
print("\n【分類の種類】")
classes = df_meta['@class'].unique()
for cls in classes:
    # 各分類の件数も表示
    count = len(df_meta[df_meta['@class'] == cls])
    print(f"  - {cls}: {count}件")

# 詳細情報も表示
print(f"\n【メタ情報の総件数】: {len(df_meta)}件")
```

---

### 解答3: データの保存

```python
"""
問題3の模範解答: GDPデータの取得と保存
"""
from jpy_datareader.data import (
    get_data_estat_statslist,
    get_data_estat_statsdata
)
from dotenv import load_dotenv
import os

load_dotenv()

# ステップ1: GDP統計を検索
print("【ステップ1】GDP統計の検索")
df_list = get_data_estat_statslist(searchWord="GDP", limit=5)
print(f"検索結果: {len(df_list)}件")

# 最初の統計表を選択
stats_id = df_list.iloc[0]['@id']
stats_title = df_list.iloc[0]['TITLE']
print(f"\n選択した統計表:")
print(f"  ID: {stats_id}")
print(f"  タイトル: {stats_title}")

# ステップ2: データを取得
print("\n【ステップ2】データの取得")
df_data = get_data_estat_statsdata(statsDataId=stats_id, limit=100)
print(f"取得データ: {len(df_data)}行 × {len(df_data.columns)}列")

# ステップ3: CSVに保存
output_file = "gdp_data_sample.csv"
df_data.to_csv(output_file, index=False, encoding='utf-8-sig')
print(f"\n【ステップ3】保存完了")
print(f"  ファイル名: {output_file}")
print(f"  ファイルサイズ: {os.path.getsize(output_file):,} bytes")

# データの先頭を表示
print("\n【データプレビュー】")
print(df_data.head())
```

---

### 解答4: エラーハンドリング

```python
"""
問題4の模範解答: エラーハンドリング
"""
from jpy_datareader.data import get_data_estat_statsdata
from dotenv import load_dotenv

load_dotenv()

def safe_get_data(stats_id):
    """
    統計データを安全に取得する関数

    Parameters:
    -----------
    stats_id : str
        統計表ID

    Returns:
    --------
    pandas.DataFrame or None
        データ取得成功時はDataFrame、失敗時はNone
    """
    try:
        # データを取得
        print(f"統計表ID {stats_id} のデータを取得中...")
        df = get_data_estat_statsdata(statsDataId=stats_id, limit=10)

        # 成功メッセージ
        print(f"✅ データ取得成功: {len(df)}件")
        return df

    except Exception as e:
        # エラーメッセージを表示
        print(f"❌ エラーが発生しました")
        print(f"   統計表ID: {stats_id}")
        print(f"   エラー内容: {type(e).__name__}")
        print(f"   詳細: {str(e)}")
        return None

# テスト1: 正常なID（実在するID）
print("【テスト1】正常なIDでの取得")
print("=" * 50)
df_valid = safe_get_data("0003410379")  # 実在するID（例）

# テスト2: 存在しないID
print("\n【テスト2】存在しないIDでの取得")
print("=" * 50)
df_invalid = safe_get_data("9999999999")  # 存在しないID

# テスト3: 不正な形式のID
print("\n【テスト3】不正な形式のIDでの取得")
print("=" * 50)
df_malformed = safe_get_data("invalid_id")  # 不正な形式
```

---

### 解答5: データの基本分析

```python
"""
問題5の模範解答: 人口データの基本分析
"""
from jpy_datareader.data import (
    get_data_estat_statslist,
    get_data_estat_statsdata
)
from dotenv import load_dotenv
import pandas as pd

load_dotenv()

# ステップ1: 人口統計を検索
print("【ステップ1】人口統計の検索")
print("=" * 50)
df_list = get_data_estat_statslist(searchWord="人口", limit=5)

# 最初の統計表を選択
stats_id = df_list.iloc[0]['@id']
stats_title = df_list.iloc[0]['TITLE']
print(f"統計表: {stats_title}")
print(f"ID: {stats_id}")

# ステップ2: データを取得
print("\n【ステップ2】データの取得")
print("=" * 50)
df_data = get_data_estat_statsdata(statsDataId=stats_id, limit=1000)
print(f"取得データ: {len(df_data)}行 × {len(df_data.columns)}列")

# ステップ3: 数値カラムを抽出
print("\n【ステップ3】数値カラムの抽出")
print("=" * 50)
numeric_cols = df_data.select_dtypes(include=['float64', 'int64']).columns
print(f"数値カラム数: {len(numeric_cols)}")

if len(numeric_cols) > 0:
    print("\n数値カラム一覧:")
    for col in numeric_cols:
        print(f"  - {col}")

    # ステップ4: 平均値を計算
    print("\n【ステップ4】平均値の計算")
    print("=" * 50)

    for col in numeric_cols:
        # 欠損値を除いて平均を計算
        mean_value = df_data[col].mean()

        # 欠損値の数も確認
        null_count = df_data[col].isnull().sum()
        valid_count = len(df_data) - null_count

        print(f"\n【{col}】")
        print(f"  平均値: {mean_value:,.2f}")
        print(f"  有効データ数: {valid_count:,}")
        print(f"  欠損値数: {null_count:,}")
        print(f"  最小値: {df_data[col].min():,.2f}")
        print(f"  最大値: {df_data[col].max():,.2f}")

    # 基本統計量をまとめて表示
    print("\n【全体の基本統計量】")
    print("=" * 50)
    print(df_data[numeric_cols].describe())

else:
    print("\n⚠️ 数値カラムが見つかりませんでした")
    print("この統計表には数値データが含まれていない可能性があります")
```

---

## 🎯 まとめ

この章では以下を学びました：

✅ JPy-DataReaderのインストール方法
✅ e-Stat APIキーの取得と設定
✅ 3つの基本関数の使い方:
  - `get_data_estat_statslist()` - 統計表の検索
  - `get_data_estat_metainfo()` - メタ情報の取得
  - `get_data_estat_statsdata()` - 統計データの取得
✅ 基本的なエラーハンドリング
✅ データの保存と基本分析

---

## 📚 次のステップ

次の章では、より詳細な機能を学びます：

**[第2章: 中級編 - StatsListReaderで統計表を探す](./chapter02_statslist.md)**

- StatsListReaderクラスの詳細
- 複雑な検索条件の指定
- フィルタリング機能の活用
- 効率的な統計表の探し方

---

## 💡 よくある質問

**Q1: APIキーが正しく設定されているか確認するには？**

```python
import os
from dotenv import load_dotenv

load_dotenv()
api_key = os.getenv('ESTAT_APP_ID')

if api_key:
    print(f"✅ APIキーが設定されています: {api_key[:5]}...")
else:
    print("❌ APIキーが設定されていません")
```

**Q2: データ取得に時間がかかる場合は？**

- `limit`パラメータで取得件数を制限してください
- 大量データの場合は自動ページネーションが動作します

**Q3: エラーが発生した場合は？**

- APIキーが正しいか確認
- インターネット接続を確認
- 統計表IDが正しいか確認
- エラーメッセージを確認してデバッグ

---

お疲れ様でした！基礎編はこれで完了です。
