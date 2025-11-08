# 第3章: 中級編 - MetaInfoReaderでメタ情報を取得

## 📚 この章で学ぶこと

- MetaInfoReaderクラスの詳細な使い方
- メタ情報とは何か、なぜ重要なのか
- データ構造の理解
- 分類階層の取得と活用
- メタデータを使った効率的なデータ取得

**対象レベル**: 中級者
**所要時間**: 約60分
**前提知識**: 第1章、第2章の内容

---

## 3.1 メタ情報とは

### 3.1.1 メタ情報の定義

メタ情報（メタデータ）とは、統計データの「構造」を説明する情報です。

**メタ情報に含まれる内容:**
- どのような分類項目があるか（例: 地域、年齢、性別）
- 各分類にどのようなコード値があるか
- 分類の階層構造（例: 全国 > 都道府県 > 市区町村）
- データの単位（人、千円、%など）

### 3.1.2 なぜメタ情報が重要なのか

```python
"""
メタ情報が重要な理由を示すサンプル
"""

# ❌ メタ情報を使わない場合
# → すべてのデータを取得してから処理
#    データ量が多い、時間がかかる、不要なデータも取得

# ✅ メタ情報を使う場合
# → 必要なデータだけを効率的に取得
#    データ量が少ない、高速、正確
```

**メタ情報を使うメリット:**

1. **効率的なデータ取得**: 必要なデータだけを絞り込める
2. **データ理解**: どのような分類があるか事前にわかる
3. **エラー回避**: 存在しないコードを指定するミスを防げる
4. **階層構造の把握**: 地域や分類の親子関係がわかる

---

## 3.2 MetaInfoReaderの基本

### 3.2.1 基本的な使い方

```python
"""
MetaInfoReaderの基本的な使い方
"""
import os
from dotenv import load_dotenv
from jpy_datareader.estat import MetaInfoReader
import pandas as pd

# 環境変数を読み込む
load_dotenv()

# MetaInfoReaderのインスタンスを作成
reader = MetaInfoReader()

# 統計表IDを指定してメタ情報を取得
# 例: 家計調査のID
stats_id = "0003410379"

# メタ情報を取得
df_meta = reader.read(statsDataId=stats_id)

# 結果を確認
print(f"メタ情報: {len(df_meta)}件")
print(f"カラム数: {len(df_meta.columns)}")

# 最初の5行を表示
print("\n【メタ情報の先頭】")
print(df_meta.head())
```

### 3.2.2 メタ情報の構造

```python
"""
メタ情報の構造を理解する
"""
from jpy_datareader.estat import MetaInfoReader
from dotenv import load_dotenv

load_dotenv()

reader = MetaInfoReader()

# メタ情報を取得
stats_id = "0003410379"
df_meta = reader.read(statsDataId=stats_id)

# ===== カラムの確認 =====
print("【利用可能なカラム】")
print("=" * 60)
for col in df_meta.columns:
    print(f"  - {col}")

# ===== 主要なカラムの説明 =====
print("\n【主要カラムの説明】")
print("=" * 60)

important_cols = {
    '@class': '分類の種類（tab, cat01, area, timeなど）',
    '@code': '分類コード（実際のデータ取得時に使用）',
    '@name': '分類名（日本語表示名）',
    '@level': '階層レベル（1が最上位）',
    '@parentCode': '親コード（階層構造の場合）'
}

for col, description in important_cols.items():
    if col in df_meta.columns:
        print(f"\n{col}")
        print(f"  説明: {description}")
        print(f"  サンプル: {df_meta[col].iloc[0]}")

# ===== 分類の種類を確認 =====
print("\n【分類の種類】")
print("=" * 60)

if '@class' in df_meta.columns:
    classes = df_meta['@class'].unique()
    for cls in classes:
        count = len(df_meta[df_meta['@class'] == cls])
        print(f"  {cls}: {count}件")
```

---

## 3.3 分類の詳細な取得

### 3.3.1 特定の分類だけを抽出

```python
"""
特定の分類だけを抽出する
"""
from jpy_datareader.estat import MetaInfoReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

def get_classification(stats_id, class_name):
    """
    特定の分類を取得

    Parameters:
    -----------
    stats_id : str
        統計表ID
    class_name : str
        分類名（例: 'area', 'time', 'cat01'）

    Returns:
    --------
    pandas.DataFrame
        指定した分類のメタ情報
    """
    reader = MetaInfoReader()

    # メタ情報を取得
    df_meta = reader.read(statsDataId=stats_id)

    # 指定した分類だけをフィルタ
    df_class = df_meta[df_meta['@class'] == class_name].copy()

    print(f"✅ 分類 '{class_name}': {len(df_class)}件")

    return df_class


# ===== 使用例 =====
stats_id = "0003410379"

# 地域分類を取得
print("【地域分類】")
print("=" * 60)
df_area = get_classification(stats_id, 'area')

if len(df_area) > 0:
    # 最初の10件を表示
    print("\n地域の例（最初の10件）:")
    for idx, row in df_area.head(10).iterrows():
        code = row['@code']
        name = row['@name']
        level = row.get('@level', 'N/A')
        print(f"  [{code}] {name} (レベル: {level})")

# 時間分類を取得
print("\n【時間分類】")
print("=" * 60)
df_time = get_classification(stats_id, 'time')

if len(df_time) > 0:
    print(f"\n時間の範囲: {len(df_time)}期間")
    print("最初の5期間:")
    for idx, row in df_time.head().iterrows():
        print(f"  [{row['@code']}] {row['@name']}")

# カテゴリ分類を取得
print("\n【カテゴリ分類（cat01）】")
print("=" * 60)
df_cat = get_classification(stats_id, 'cat01')

if len(df_cat) > 0:
    print(f"\nカテゴリ数: {len(df_cat)}件")
    print("最初の10カテゴリ:")
    for idx, row in df_cat.head(10).iterrows():
        print(f"  [{row['@code']}] {row['@name']}")
```

### 3.3.2 階層構造の理解

```python
"""
階層構造を持つ分類の処理
"""
from jpy_datareader.estat import MetaInfoReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

def show_hierarchy(stats_id, class_name='area'):
    """
    階層構造を持つ分類を階層別に表示

    Parameters:
    -----------
    stats_id : str
        統計表ID
    class_name : str
        分類名（デフォルト: 'area'）
    """
    reader = MetaInfoReader()

    # メタ情報を取得
    df_meta = reader.read(statsDataId=stats_id)

    # 指定した分類を抽出
    df_class = df_meta[df_meta['@class'] == class_name].copy()

    # @levelカラムがあるか確認
    if '@level' not in df_class.columns:
        print(f"⚠️ '{class_name}'には階層情報がありません")
        return

    # レベルを数値型に変換
    df_class['@level'] = pd.to_numeric(df_class['@level'], errors='coerce')

    # レベル別に表示
    levels = sorted(df_class['@level'].dropna().unique())

    print(f"【{class_name}の階層構造】")
    print("=" * 60)
    print(f"階層数: {len(levels)}レベル")

    for level in levels:
        df_level = df_class[df_class['@level'] == level]

        print(f"\n▼ レベル {int(level)} ({len(df_level)}件)")
        print("-" * 60)

        # 最初の5件を表示
        for idx, row in df_level.head(5).iterrows():
            indent = "  " * int(level)  # インデントでレベルを表現
            code = row['@code']
            name = row['@name']

            # 親コードがあれば表示
            parent = row.get('@parentCode', '')
            if parent:
                print(f"{indent}[{code}] {name} (親: {parent})")
            else:
                print(f"{indent}[{code}] {name}")

        # 件数が多い場合は省略
        if len(df_level) > 5:
            print(f"{' ' * len(indent)}... 他 {len(df_level) - 5}件")


# 使用例
stats_id = "0003410379"
show_hierarchy(stats_id, 'area')
```

---

## 3.4 メタ情報を活用したデータ取得

### 3.4.1 メタ情報から必要なコードを特定

```python
"""
メタ情報から必要なコードを特定してデータ取得
"""
from jpy_datareader.estat import MetaInfoReader, StatsDataReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

def find_codes_by_name(stats_id, class_name, search_keyword):
    """
    名前でコードを検索

    Parameters:
    -----------
    stats_id : str
        統計表ID
    class_name : str
        分類名
    search_keyword : str
        検索キーワード

    Returns:
    --------
    list
        該当するコードのリスト
    """
    # メタ情報を取得
    meta_reader = MetaInfoReader()
    df_meta = meta_reader.read(statsDataId=stats_id)

    # 指定した分類を抽出
    df_class = df_meta[df_meta['@class'] == class_name].copy()

    # キーワードで検索
    df_found = df_class[df_class['@name'].str.contains(search_keyword, na=False)]

    # コードのリストを作成
    codes = df_found['@code'].tolist()

    print(f"✅ '{search_keyword}'に該当するコード: {len(codes)}件")

    # 該当項目を表示
    for idx, row in df_found.iterrows():
        print(f"  [{row['@code']}] {row['@name']}")

    return codes


# ===== 実践例: 東京都のデータだけを取得 =====
print("【実践例】東京都のデータを取得")
print("=" * 60)

stats_id = "0003410379"

# ステップ1: 地域コードから「東京都」を検索
print("\nステップ1: 東京都のコードを検索")
tokyo_codes = find_codes_by_name(stats_id, 'area', '東京都')

# ステップ2: そのコードを使ってデータ取得
if len(tokyo_codes) > 0:
    print("\nステップ2: データを取得")

    # 最初のコードを使用
    tokyo_code = tokyo_codes[0]

    data_reader = StatsDataReader()
    df_data = data_reader.read(
        statsDataId=stats_id,
        cdArea=tokyo_code,  # 東京都のコードを指定
        limit=100
    )

    print(f"✅ {len(df_data)}件のデータを取得しました")
    print("\nデータの先頭:")
    print(df_data.head())
```

### 3.4.2 複数のコードを組み合わせてデータ取得

```python
"""
複数のコードを組み合わせたデータ取得
"""
from jpy_datareader.estat import MetaInfoReader, StatsDataReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

def get_filtered_data(
    stats_id,
    area_keywords=None,
    time_keywords=None,
    limit=1000
):
    """
    メタ情報を使って絞り込んだデータを取得

    Parameters:
    -----------
    stats_id : str
        統計表ID
    area_keywords : str, optional
        地域のキーワード
    time_keywords : str, optional
        時間のキーワード
    limit : int
        取得件数

    Returns:
    --------
    pandas.DataFrame
        取得したデータ
    """
    meta_reader = MetaInfoReader()
    df_meta = meta_reader.read(statsDataId=stats_id)

    # データ取得用のパラメータを構築
    params = {
        'statsDataId': stats_id,
        'limit': limit
    }

    # 地域コードを検索
    if area_keywords:
        df_area = df_meta[df_meta['@class'] == 'area']
        df_area_found = df_area[df_area['@name'].str.contains(area_keywords, na=False)]

        if len(df_area_found) > 0:
            # 最初のコードを使用
            params['cdArea'] = df_area_found.iloc[0]['@code']
            print(f"✅ 地域: {df_area_found.iloc[0]['@name']} [{params['cdArea']}]")

    # 時間コードを検索
    if time_keywords:
        df_time = df_meta[df_meta['@class'] == 'time']
        df_time_found = df_time[df_time['@name'].str.contains(time_keywords, na=False)]

        if len(df_time_found) > 0:
            # 最初のコードを使用
            params['cdTime'] = df_time_found.iloc[0]['@code']
            print(f"✅ 時間: {df_time_found.iloc[0]['@name']} [{params['cdTime']}]")

    # データを取得
    data_reader = StatsDataReader()
    df_data = data_reader.read(**params)

    print(f"\n✅ {len(df_data)}件のデータを取得しました")

    return df_data


# ===== 使用例 =====
print("【使用例】大阪府の2024年データを取得")
print("=" * 60)

stats_id = "0003410379"

df = get_filtered_data(
    stats_id,
    area_keywords="大阪府",
    time_keywords="2024",
    limit=500
)

if df is not None and len(df) > 0:
    print("\nデータの先頭:")
    print(df.head())
```

---

## 3.5 メタ情報の分析と可視化

### 3.5.1 メタ情報の統計分析

```python
"""
メタ情報の統計分析
"""
from jpy_datareader.estat import MetaInfoReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

class MetaInfoAnalyzer:
    """
    メタ情報を分析するクラス
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
        self.reader = MetaInfoReader()
        self.df_meta = None

        # メタ情報を取得
        self.load_metadata()

    def load_metadata(self):
        """メタ情報を読み込む"""
        self.df_meta = self.reader.read(statsDataId=self.stats_id)
        print(f"✅ メタ情報を読み込みました: {len(self.df_meta)}件")

    def show_summary(self):
        """メタ情報のサマリーを表示"""
        print("\n" + "=" * 60)
        print("【メタ情報サマリー】")
        print("=" * 60)

        # 総件数
        print(f"\n総件数: {len(self.df_meta)}件")

        # 分類別件数
        print("\n【分類別件数】")
        class_counts = self.df_meta['@class'].value_counts()
        for cls, count in class_counts.items():
            print(f"  {cls:15} : {count:5}件")

    def analyze_classification(self, class_name):
        """
        特定の分類を詳しく分析

        Parameters:
        -----------
        class_name : str
            分類名
        """
        print(f"\n" + "=" * 60)
        print(f"【{class_name}の詳細分析】")
        print("=" * 60)

        # 分類を抽出
        df_class = self.df_meta[self.df_meta['@class'] == class_name].copy()

        if len(df_class) == 0:
            print(f"⚠️ '{class_name}'は存在しません")
            return

        print(f"\n総件数: {len(df_class)}件")

        # レベルがあるか確認
        if '@level' in df_class.columns:
            df_class['@level'] = pd.to_numeric(df_class['@level'], errors='coerce')
            levels = df_class['@level'].dropna()

            if len(levels) > 0:
                print(f"\n【階層情報】")
                print(f"  最小レベル: {int(levels.min())}")
                print(f"  最大レベル: {int(levels.max())}")

                # レベル別件数
                print(f"\n  レベル別件数:")
                level_counts = df_class['@level'].value_counts().sort_index()
                for level, count in level_counts.items():
                    if pd.notna(level):
                        print(f"    レベル {int(level)}: {count}件")

        # サンプルを表示
        print(f"\n【サンプル（最初の10件）】")
        for idx, row in df_class.head(10).iterrows():
            code = row['@code']
            name = row['@name']
            print(f"  [{code}] {name}")

    def export_classification(self, class_name, filename=None):
        """
        特定の分類をCSVに出力

        Parameters:
        -----------
        class_name : str
            分類名
        filename : str, optional
            出力ファイル名
        """
        # 分類を抽出
        df_class = self.df_meta[self.df_meta['@class'] == class_name].copy()

        if len(df_class) == 0:
            print(f"⚠️ '{class_name}'は存在しません")
            return

        # ファイル名を生成
        if filename is None:
            filename = f"{self.stats_id}_{class_name}.csv"

        # 保存
        df_class.to_csv(filename, index=False, encoding='utf-8-sig')
        print(f"✅ {class_name}を {filename} に保存しました（{len(df_class)}件）")


# ===== 使用例 =====
print("【メタ情報の分析】")
print("=" * 60)

stats_id = "0003410379"

# Analyzerのインスタンスを作成
analyzer = MetaInfoAnalyzer(stats_id)

# サマリーを表示
analyzer.show_summary()

# 地域分類を分析
analyzer.analyze_classification('area')

# 時間分類を分析
analyzer.analyze_classification('time')

# 地域分類をCSVに出力
analyzer.export_classification('area', 'area_codes.csv')
```

---

## 📝 練習問題

### 問題1: 基本的なメタ情報取得

**課題**: 統計表ID "0003410379" のメタ情報を取得し、含まれる分類の種類とそれぞれの件数を表示してください。

<details>
<summary>ヒント</summary>

- `MetaInfoReader()`でインスタンス作成
- `read(statsDataId=...)`でメタ情報取得
- `value_counts()`で分類別件数を集計

</details>

---

### 問題2: 特定地域のコード検索

**課題**: メタ情報から「北海道」に関する地域コードをすべて検索し、コードと名前を表示してください。

<details>
<summary>ヒント</summary>

- 分類名は'area'
- `str.contains('北海道')`でフィルタ
- '@code'と'@name'を表示

</details>

---

### 問題3: 階層構造の可視化

**課題**: 地域分類の階層構造を分析し、各レベルに何件の地域があるか表示してください。また、レベル1（最上位）の地域をすべて表示してください。

<details>
<summary>ヒント</summary>

- '@level'カラムを使用
- `value_counts().sort_index()`でレベル別集計
- レベル1をフィルタして表示

</details>

---

### 問題4: コードを使ったデータ取得

**課題**: メタ情報から「神奈川県」のコードを検索し、そのコードを使って実際のデータを100件取得してください。

<details>
<summary>ヒント</summary>

- MetaInfoReaderで地域コードを検索
- StatsDataReaderでデータ取得
- `cdArea`パラメータにコードを指定

</details>

---

### 問題5: メタ情報の比較

**課題**: 2つの異なる統計表のメタ情報を取得し、それぞれの分類構造を比較してください。共通する分類と固有の分類を表示してください。

<details>
<summary>ヒント</summary>

- 2つの統計表IDでメタ情報を取得
- '@class'のuniqueで分類リストを取得
- set()で共通要素と差分を計算

</details>

---

## 📖 模範解答

### 解答1: 基本的なメタ情報取得

```python
"""
問題1の模範解答: メタ情報の基本取得
"""
from jpy_datareader.estat import MetaInfoReader
from dotenv import load_dotenv

load_dotenv()

# MetaInfoReaderのインスタンスを作成
reader = MetaInfoReader()

# 統計表IDを指定
stats_id = "0003410379"

# メタ情報を取得
df_meta = reader.read(statsDataId=stats_id)

print(f"【統計表ID: {stats_id}】")
print("=" * 60)
print(f"総メタ情報件数: {len(df_meta)}件")
print(f"カラム数: {len(df_meta.columns)}")

# 分類の種類と件数を表示
print("\n【分類の種類と件数】")
print("=" * 60)

class_counts = df_meta['@class'].value_counts()

# 見やすく表示
for cls, count in class_counts.items():
    # パーセンテージも計算
    percentage = (count / len(df_meta)) * 100

    # バーグラフ風に表示
    bar = "■" * (count // 10)

    print(f"{cls:15} : {count:5}件 ({percentage:5.1f}%) {bar}")

# 総計を確認
print("=" * 60)
print(f"{'合計':15} : {class_counts.sum():5}件")
```

---

### 解答2: 特定地域のコード検索

```python
"""
問題2の模範解答: 北海道コードの検索
"""
from jpy_datareader.estat import MetaInfoReader
from dotenv import load_dotenv

load_dotenv()

reader = MetaInfoReader()
stats_id = "0003410379"

# メタ情報を取得
df_meta = reader.read(statsDataId=stats_id)

# 地域分類を抽出
df_area = df_meta[df_meta['@class'] == 'area'].copy()

# 「北海道」を含む地域を検索
df_hokkaido = df_area[df_area['@name'].str.contains('北海道', na=False)]

print("【北海道に関する地域コード】")
print("=" * 60)
print(f"該当件数: {len(df_hokkaido)}件\n")

# 結果を表示
for idx, row in df_hokkaido.iterrows():
    code = row['@code']
    name = row['@name']

    # レベル情報があれば表示
    level = row.get('@level', 'N/A')

    # 親コードがあれば表示
    parent = row.get('@parentCode', '')

    if parent:
        print(f"[{code}] {name}")
        print(f"  └ レベル: {level}, 親コード: {parent}")
    else:
        print(f"[{code}] {name}")
        print(f"  └ レベル: {level}")

    print()

# 補足: コードリストをエクスポート
print("\n【コード一覧（CSV形式）】")
print("=" * 60)
print("コード,名前,レベル,親コード")
for idx, row in df_hokkaido.iterrows():
    code = row['@code']
    name = row['@name']
    level = row.get('@level', '')
    parent = row.get('@parentCode', '')
    print(f"{code},{name},{level},{parent}")
```

---

### 解答3: 階層構造の可視化

```python
"""
問題3の模範解答: 階層構造の分析
"""
from jpy_datareader.estat import MetaInfoReader
import pandas as pd
from dotenv import load_dotenv

load_dotenv()

reader = MetaInfoReader()
stats_id = "0003410379"

# メタ情報を取得
df_meta = reader.read(statsDataId=stats_id)

# 地域分類を抽出
df_area = df_meta[df_meta['@class'] == 'area'].copy()

print("【地域分類の階層構造分析】")
print("=" * 60)

# @levelカラムがあるか確認
if '@level' not in df_area.columns:
    print("⚠️ この統計表には階層情報がありません")
else:
    # レベルを数値型に変換
    df_area['@level'] = pd.to_numeric(df_area['@level'], errors='coerce')

    # レベル別の件数を集計
    level_counts = df_area['@level'].value_counts().sort_index()

    print("\n【レベル別件数】")
    print("-" * 60)

    total = 0
    for level, count in level_counts.items():
        if pd.notna(level):
            # バーグラフ風に表示
            bar = "□" * (count // 5)
            print(f"レベル {int(level):2} : {count:4}件 {bar}")
            total += count

    print("-" * 60)
    print(f"{'合計':10} : {total:4}件")

    # レベル1（最上位）の地域を表示
    print("\n【レベル1（最上位）の地域一覧】")
    print("=" * 60)

    df_level1 = df_area[df_area['@level'] == 1].copy()

    if len(df_level1) > 0:
        print(f"レベル1の地域数: {len(df_level1)}件\n")

        for idx, row in df_level1.iterrows():
            code = row['@code']
            name = row['@name']
            print(f"  [{code}] {name}")
    else:
        print("レベル1の地域はありません")

    # レベル1とレベル2の関係を表示（サンプル）
    print("\n【階層関係のサンプル（レベル1→レベル2）】")
    print("=" * 60)

    df_level2 = df_area[df_area['@level'] == 2].copy()

    if len(df_level2) > 0 and '@parentCode' in df_level2.columns:
        # 最初のレベル1地域を選択
        if len(df_level1) > 0:
            parent_code = df_level1.iloc[0]['@code']
            parent_name = df_level1.iloc[0]['@name']

            print(f"\n親: [{parent_code}] {parent_name}")
            print("  子地域:")

            # 子地域を抽出
            df_children = df_level2[df_level2['@parentCode'] == parent_code]

            for idx, row in df_children.head(10).iterrows():
                print(f"    └ [{row['@code']}] {row['@name']}")

            if len(df_children) > 10:
                print(f"    ... 他 {len(df_children) - 10}件")
```

---

### 解答4: コードを使ったデータ取得

```python
"""
問題4の模範解答: メタ情報を使ったデータ取得
"""
from jpy_datareader.estat import MetaInfoReader, StatsDataReader
from dotenv import load_dotenv

load_dotenv()

print("【神奈川県のデータ取得】")
print("=" * 60)

stats_id = "0003410379"

# ===== ステップ1: メタ情報から神奈川県のコードを検索 =====
print("\nステップ1: 神奈川県のコードを検索")
print("-" * 60)

meta_reader = MetaInfoReader()
df_meta = meta_reader.read(statsDataId=stats_id)

# 地域分類を抽出
df_area = df_meta[df_meta['@class'] == 'area'].copy()

# 神奈川県を検索
df_kanagawa = df_area[df_area['@name'].str.contains('神奈川県', na=False)]

if len(df_kanagawa) == 0:
    print("❌ 神奈川県が見つかりませんでした")
else:
    print(f"✅ {len(df_kanagawa)}件の該当地域が見つかりました\n")

    # 該当地域を表示
    for idx, row in df_kanagawa.iterrows():
        code = row['@code']
        name = row['@name']
        level = row.get('@level', 'N/A')
        print(f"  [{code}] {name} (レベル: {level})")

    # ===== ステップ2: コードを使ってデータ取得 =====
    print("\nステップ2: データを取得")
    print("-" * 60)

    # 最初のコードを使用
    kanagawa_code = df_kanagawa.iloc[0]['@code']
    kanagawa_name = df_kanagawa.iloc[0]['@name']

    print(f"使用するコード: [{kanagawa_code}] {kanagawa_name}")

    # データを取得
    data_reader = StatsDataReader()
    df_data = data_reader.read(
        statsDataId=stats_id,
        cdArea=kanagawa_code,  # 神奈川県のコード
        limit=100              # 100件取得
    )

    # ===== ステップ3: 取得したデータを確認 =====
    print("\nステップ3: データ確認")
    print("-" * 60)

    print(f"✅ {len(df_data)}件のデータを取得しました")
    print(f"カラム数: {len(df_data.columns)}")

    # データの先頭を表示
    print("\n【データの先頭5行】")
    print(df_data.head())

    # 数値カラムがあれば統計量を表示
    numeric_cols = df_data.select_dtypes(include=['float64', 'int64']).columns

    if len(numeric_cols) > 0:
        print("\n【基本統計量】")
        print(df_data[numeric_cols].describe())

    # CSVに保存
    output_file = f"kanagawa_data_{stats_id}.csv"
    df_data.to_csv(output_file, index=False, encoding='utf-8-sig')
    print(f"\n✅ データを {output_file} に保存しました")
```

---

### 解答5: メタ情報の比較

```python
"""
問題5の模範解答: 複数統計表のメタ情報比較
"""
from jpy_datareader.estat import MetaInfoReader
from dotenv import load_dotenv

load_dotenv()

def get_classifications(stats_id):
    """
    統計表の分類リストを取得

    Parameters:
    -----------
    stats_id : str
        統計表ID

    Returns:
    --------
    set
        分類名のセット
    """
    reader = MetaInfoReader()
    df_meta = reader.read(statsDataId=stats_id)

    classifications = set(df_meta['@class'].unique())

    return classifications


print("【複数統計表のメタ情報比較】")
print("=" * 60)

# 2つの統計表を比較
stats_id1 = "0003410379"  # 家計調査
stats_id2 = "0003348423"  # 別の統計表（例）

# ===== ステップ1: 各統計表の分類を取得 =====
print("\nステップ1: 各統計表の分類を取得")
print("-" * 60)

try:
    classes1 = get_classifications(stats_id1)
    print(f"\n統計表1 [{stats_id1}]")
    print(f"分類数: {len(classes1)}種類")
    print(f"分類: {', '.join(sorted(classes1))}")

except Exception as e:
    print(f"❌ 統計表1のエラー: {e}")
    classes1 = set()

try:
    classes2 = get_classifications(stats_id2)
    print(f"\n統計表2 [{stats_id2}]")
    print(f"分類数: {len(classes2)}種類")
    print(f"分類: {', '.join(sorted(classes2))}")

except Exception as e:
    print(f"❌ 統計表2のエラー: {e}")
    classes2 = set()

# ===== ステップ2: 共通分類と固有分類を抽出 =====
print("\n\nステップ2: 分類の比較")
print("=" * 60)

# 共通分類
common_classes = classes1 & classes2
print(f"\n【共通する分類】({len(common_classes)}種類)")
if common_classes:
    for cls in sorted(common_classes):
        print(f"  ✓ {cls}")
else:
    print("  なし")

# 統計表1のみの分類
unique_classes1 = classes1 - classes2
print(f"\n【統計表1のみの分類】({len(unique_classes1)}種類)")
if unique_classes1:
    for cls in sorted(unique_classes1):
        print(f"  • {cls}")
else:
    print("  なし")

# 統計表2のみの分類
unique_classes2 = classes2 - classes1
print(f"\n【統計表2のみの分類】({len(unique_classes2)}種類)")
if unique_classes2:
    for cls in sorted(unique_classes2):
        print(f"  • {cls}")
else:
    print("  なし")

# ===== ステップ3: ベン図風の可視化 =====
print("\n\nステップ3: 視覚的な比較")
print("=" * 60)

print(f"""
┌──────────────────────────────────────┐
│         分類の分布図                 │
├──────────────────────────────────────┤
│ 統計表1のみ       : {len(unique_classes1):3}種類         │
│ 統計表2のみ       : {len(unique_classes2):3}種類         │
│ 共通              : {len(common_classes):3}種類         │
├──────────────────────────────────────┤
│ 統計表1合計       : {len(classes1):3}種類         │
│ 統計表2合計       : {len(classes2):3}種類         │
│ 全体（和集合）    : {len(classes1 | classes2):3}種類         │
└──────────────────────────────────────┘
""")

# ===== 補足: 詳細な分類内容の比較 =====
print("\n【補足】共通分類の詳細比較")
print("=" * 60)

if len(common_classes) > 0:
    reader = MetaInfoReader()

    for cls in sorted(list(common_classes)[:3]):  # 最初の3つだけ
        print(f"\n▼ 分類: {cls}")

        try:
            # 統計表1
            df_meta1 = reader.read(statsDataId=stats_id1)
            count1 = len(df_meta1[df_meta1['@class'] == cls])
            print(f"  統計表1: {count1}件")

            # 統計表2
            df_meta2 = reader.read(statsDataId=stats_id2)
            count2 = len(df_meta2[df_meta2['@class'] == cls])
            print(f"  統計表2: {count2}件")

        except Exception as e:
            print(f"  エラー: {e}")
```

---

## 🎯 まとめ

この章では以下を学びました：

✅ MetaInfoReaderクラスの詳細な使い方
✅ メタ情報の構造と重要性
✅ 分類の抽出とフィルタリング
✅ 階層構造の理解と分析
✅ メタ情報を活用した効率的なデータ取得
✅ 複数統計表のメタ情報比較

---

## 📚 次のステップ

次の章では、実際のデータ取得について詳しく学びます：

**[第4章: 中級編 - StatsDataReaderでデータを取得](./chapter04_statsdata.md)**

- StatsDataReaderクラスの詳細
- 大量データの効率的な取得
- 様々なフィルタリングオプション
- データの前処理テクニック

---

お疲れ様でした！
