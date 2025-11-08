# 第6章: 上級編 - エラーハンドリングとベストプラクティス

## 📚 この章で学ぶこと

- エラーハンドリングの詳細
- リトライロジックの実装
- パフォーマンス最適化
- データキャッシュ戦略
- セキュリティのベストプラクティス
- 本番環境での運用

**対象レベル**: 上級者
**所要時間**: 約60分
**前提知識**: 第1〜5章の内容、Pythonの例外処理

---

## 6.1 エラーハンドリング

### 6.1.1 基本的なエラー処理

```python
"""
基本的なエラーハンドリング
"""
from jpy_datareader.estat import StatsDataReader
from dotenv import load_dotenv
import time

load_dotenv()

def safe_data_fetch(stats_id, **kwargs):
    """
    安全にデータを取得する関数

    Parameters:
    -----------
    stats_id : str
        統計表ID
    **kwargs : dict
        その他のパラメータ

    Returns:
    --------
    tuple : (success, data_or_error)
        success: bool - 成功したかどうか
        data_or_error: DataFrame or Exception - データまたはエラー
    """
    reader = StatsDataReader()

    try:
        # データを取得
        df = reader.read(statsDataId=stats_id, **kwargs)

        # 成功
        return True, df

    except ValueError as e:
        # パラメータエラー
        print(f"❌ パラメータエラー: {e}")
        return False, e

    except ConnectionError as e:
        # 接続エラー
        print(f"❌ 接続エラー: {e}")
        return False, e

    except TimeoutError as e:
        # タイムアウト
        print(f"❌ タイムアウト: {e}")
        return False, e

    except Exception as e:
        # その他のエラー
        print(f"❌ 予期しないエラー: {type(e).__name__}: {e}")
        return False, e


# 使用例
print("【安全なデータ取得】")
print("=" * 60)

stats_id = "0003410379"

success, result = safe_data_fetch(stats_id, limit=100)

if success:
    print(f"✅ 成功: {len(result)}件のデータを取得")
else:
    print(f"❌ 失敗: {result}")
```

### 6.1.2 リトライロジックの実装

```python
"""
リトライロジック付きのデータ取得
"""
from jpy_datareader.estat import StatsDataReader
from dotenv import load_dotenv
import time

load_dotenv()

def fetch_with_retry(
    stats_id,
    max_retries=3,
    retry_delay=2,
    backoff_factor=2,
    **kwargs
):
    """
    リトライ機能付きのデータ取得

    Parameters:
    -----------
    stats_id : str
        統計表ID
    max_retries : int
        最大リトライ回数
    retry_delay : float
        初期リトライ待機時間（秒）
    backoff_factor : float
        待機時間の増加率
    **kwargs : dict
        その他のパラメータ

    Returns:
    --------
    pandas.DataFrame or None
        取得したデータ、失敗時はNone
    """
    reader = StatsDataReader()

    for attempt in range(max_retries + 1):
        try:
            print(f"\n試行 {attempt + 1}/{max_retries + 1}...")

            # データを取得
            df = reader.read(statsDataId=stats_id, **kwargs)

            # 成功
            print(f"✅ 成功: {len(df)}件のデータを取得")
            return df

        except Exception as e:
            print(f"❌ エラー: {type(e).__name__}: {e}")

            # 最後の試行なら諦める
            if attempt == max_retries:
                print(f"\n❌ {max_retries + 1}回試行しましたが失敗しました")
                return None

            # 待機時間を計算（指数バックオフ）
            wait_time = retry_delay * (backoff_factor ** attempt)
            print(f"⏳ {wait_time:.1f}秒待機してリトライします...")

            time.sleep(wait_time)

    return None


# 使用例
print("【リトライ機能付きデータ取得】")
print("=" * 60)

stats_id = "0003410379"

df = fetch_with_retry(
    stats_id,
    limit=100,
    max_retries=3,
    retry_delay=1,
    backoff_factor=2
)

if df is not None:
    print(f"\n最終結果: {len(df)}件のデータを取得しました")
else:
    print("\n最終結果: データ取得に失敗しました")
```

---

## 6.2 パフォーマンス最適化

### 6.2.1 バッチ処理

```python
"""
効率的なバッチ処理
"""
from jpy_datareader.estat import StatsDataReader
from dotenv import load_dotenv
import pandas as pd
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

load_dotenv()

class BatchDataFetcher:
    """
    バッチでデータを取得するクラス
    """

    def __init__(self, max_workers=3):
        """
        初期化

        Parameters:
        -----------
        max_workers : int
            並列処理の最大ワーカー数
        """
        self.max_workers = max_workers
        self.reader = StatsDataReader()

    def fetch_single(self, stats_id, **kwargs):
        """
        単一の統計表を取得

        Parameters:
        -----------
        stats_id : str
            統計表ID
        **kwargs : dict
            その他のパラメータ

        Returns:
        --------
        tuple : (stats_id, DataFrame or None)
        """
        try:
            print(f"  [{stats_id}] 取得開始...")
            start_time = time.time()

            df = self.reader.read(statsDataId=stats_id, **kwargs)

            elapsed = time.time() - start_time
            print(f"  [{stats_id}] 完了 ({elapsed:.2f}秒, {len(df)}件)")

            return stats_id, df

        except Exception as e:
            print(f"  [{stats_id}] エラー: {e}")
            return stats_id, None

    def fetch_batch(self, stats_ids, **kwargs):
        """
        複数の統計表を並列で取得

        Parameters:
        -----------
        stats_ids : list
            統計表IDのリスト
        **kwargs : dict
            各統計表に共通のパラメータ

        Returns:
        --------
        dict : {stats_id: DataFrame}
        """
        print(f"【バッチ取得開始】")
        print(f"統計表数: {len(stats_ids)}")
        print(f"並列数: {self.max_workers}")
        print("=" * 60)

        results = {}
        start_time = time.time()

        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            # 全てのタスクを送信
            futures = {
                executor.submit(self.fetch_single, sid, **kwargs): sid
                for sid in stats_ids
            }

            # 完了したタスクから順に処理
            for future in as_completed(futures):
                stats_id, df = future.result()
                if df is not None:
                    results[stats_id] = df

        elapsed = time.time() - start_time

        print("\n" + "=" * 60)
        print(f"【バッチ取得完了】")
        print(f"成功: {len(results)}/{len(stats_ids)}件")
        print(f"総所要時間: {elapsed:.2f}秒")
        print(f"平均: {elapsed/len(stats_ids):.2f}秒/統計表")

        return results


# 使用例
print("【バッチ処理の例】")
print("=" * 60)

# 複数の統計表IDを準備
stats_ids = [
    "0003410379",
    "0003348423",
    # 他の統計表IDを追加
]

fetcher = BatchDataFetcher(max_workers=3)
results = fetcher.fetch_batch(stats_ids, limit=100)

# 結果を確認
for stats_id, df in results.items():
    print(f"\n統計表 {stats_id}: {len(df)}件")
```

---

## 6.3 データキャッシュ戦略

### 6.3.1 シンプルなキャッシュ実装

```python
"""
データキャッシュの実装
"""
from jpy_datareader.estat import StatsDataReader
from dotenv import load_dotenv
import pandas as pd
import pickle
import os
from datetime import datetime, timedelta
import hashlib

load_dotenv()

class CachedDataReader:
    """
    キャッシュ機能付きのデータリーダー
    """

    def __init__(self, cache_dir="./cache", cache_expiry_hours=24):
        """
        初期化

        Parameters:
        -----------
        cache_dir : str
            キャッシュディレクトリ
        cache_expiry_hours : int
            キャッシュの有効期限（時間）
        """
        self.cache_dir = cache_dir
        self.cache_expiry = timedelta(hours=cache_expiry_hours)
        self.reader = StatsDataReader()

        # キャッシュディレクトリを作成
        os.makedirs(cache_dir, exist_ok=True)

    def _get_cache_key(self, stats_id, **kwargs):
        """
        キャッシュキーを生成

        Parameters:
        -----------
        stats_id : str
            統計表ID
        **kwargs : dict
            その他のパラメータ

        Returns:
        --------
        str : キャッシュキー
        """
        # パラメータを文字列に変換
        params_str = f"{stats_id}_{str(sorted(kwargs.items()))}"

        # ハッシュ化
        key = hashlib.md5(params_str.encode()).hexdigest()

        return key

    def _get_cache_path(self, cache_key):
        """キャッシュファイルのパスを取得"""
        return os.path.join(self.cache_dir, f"{cache_key}.pkl")

    def _is_cache_valid(self, cache_path):
        """
        キャッシュが有効かチェック

        Parameters:
        -----------
        cache_path : str
            キャッシュファイルのパス

        Returns:
        --------
        bool : 有効ならTrue
        """
        if not os.path.exists(cache_path):
            return False

        # ファイルの更新日時を取得
        mtime = datetime.fromtimestamp(os.path.getmtime(cache_path))

        # 有効期限をチェック
        return (datetime.now() - mtime) < self.cache_expiry

    def read(self, stats_id, use_cache=True, **kwargs):
        """
        データを取得（キャッシュを使用）

        Parameters:
        -----------
        stats_id : str
            統計表ID
        use_cache : bool
            キャッシュを使用するか
        **kwargs : dict
            その他のパラメータ

        Returns:
        --------
        pandas.DataFrame
        """
        cache_key = self._get_cache_key(stats_id, **kwargs)
        cache_path = self._get_cache_path(cache_key)

        # キャッシュをチェック
        if use_cache and self._is_cache_valid(cache_path):
            print(f"✅ キャッシュから読み込み: {cache_key}")

            with open(cache_path, 'rb') as f:
                df = pickle.load(f)

            return df

        # APIから取得
        print(f"🌐 APIから取得: {stats_id}")
        df = self.reader.read(statsDataId=stats_id, **kwargs)

        # キャッシュに保存
        if use_cache:
            with open(cache_path, 'wb') as f:
                pickle.dump(df, f)

            print(f"💾 キャッシュに保存: {cache_key}")

        return df

    def clear_cache(self, older_than_hours=None):
        """
        キャッシュをクリア

        Parameters:
        -----------
        older_than_hours : int, optional
            指定時間より古いキャッシュのみ削除
        """
        print(f"【キャッシュクリア】")
        print("=" * 60)

        cleared = 0

        for filename in os.listdir(self.cache_dir):
            if not filename.endswith('.pkl'):
                continue

            filepath = os.path.join(self.cache_dir, filename)

            # 古いキャッシュのみ削除する場合
            if older_than_hours is not None:
                mtime = datetime.fromtimestamp(os.path.getmtime(filepath))
                age = datetime.now() - mtime

                if age < timedelta(hours=older_than_hours):
                    continue

            # 削除
            os.remove(filepath)
            cleared += 1

        print(f"✅ {cleared}件のキャッシュを削除しました")


# 使用例
print("【キャッシュ付きデータ取得】")
print("=" * 60)

# キャッシュリーダーを作成
cached_reader = CachedDataReader(
    cache_dir="./data_cache",
    cache_expiry_hours=24
)

stats_id = "0003410379"

# 1回目: APIから取得
print("\n1回目の取得:")
df1 = cached_reader.read(stats_id, limit=100)
print(f"データ件数: {len(df1)}")

# 2回目: キャッシュから取得
print("\n2回目の取得:")
df2 = cached_reader.read(stats_id, limit=100)
print(f"データ件数: {len(df2)}")

# キャッシュをクリア
print("\n")
cached_reader.clear_cache(older_than_hours=0)  # すべてクリア
```

---

## 6.4 セキュリティのベストプラクティス

### 6.4.1 APIキーの安全な管理

```python
"""
APIキーの安全な管理
"""
import os
from dotenv import load_dotenv
from pathlib import Path

# ❌ 悪い例: コードに直接記述
# BAD_API_KEY = "your_api_key_here"  # 絶対にやらない！

# ✅ 良い例: 環境変数を使用
load_dotenv()
api_key = os.getenv('ESTAT_APP_ID')

if not api_key:
    raise ValueError("❌ APIキーが設定されていません。.envファイルを確認してください。")

print("✅ APIキーが正しく読み込まれました")


# .envファイルの検証
def validate_env_file():
    """
    .envファイルが正しく設定されているか検証
    """
    print("【環境変数の検証】")
    print("=" * 60)

    # .envファイルの存在確認
    env_path = Path('.env')

    if not env_path.exists():
        print("❌ .envファイルが見つかりません")
        print("\n.envファイルを作成してください:")
        print("  ESTAT_APP_ID=your_app_id_here")
        return False

    print("✅ .envファイルが存在します")

    # .gitignoreの確認
    gitignore_path = Path('.gitignore')

    if gitignore_path.exists():
        with open(gitignore_path, 'r') as f:
            content = f.read()

        if '.env' in content:
            print("✅ .gitignoreに.envが含まれています")
        else:
            print("⚠️ .gitignoreに.envを追加してください！")
    else:
        print("⚠️ .gitignoreファイルが見つかりません")
        print("   .envファイルをGitにコミットしないよう注意してください")

    # APIキーの読み込み確認
    load_dotenv()
    api_key = os.getenv('ESTAT_APP_ID')

    if api_key:
        # APIキーの一部だけ表示
        masked_key = api_key[:5] + "..." + api_key[-3:] if len(api_key) > 8 else "***"
        print(f"✅ APIキーが読み込まれました: {masked_key}")
        return True
    else:
        print("❌ APIキーが読み込めません")
        return False


# 検証を実行
validate_env_file()
```

### 6.4.2 データの安全な保存

```python
"""
データの安全な保存
"""
import pandas as pd
import os
from pathlib import Path

class SecureDataSaver:
    """
    データを安全に保存するクラス
    """

    def __init__(self, output_dir="./output"):
        """
        初期化

        Parameters:
        -----------
        output_dir : str
            出力ディレクトリ
        """
        self.output_dir = Path(output_dir)

        # ディレクトリを作成（存在しない場合）
        self.output_dir.mkdir(parents=True, exist_ok=True)

        # パーミッションを設定（Unixシステムのみ）
        if os.name != 'nt':  # Windows以外
            os.chmod(self.output_dir, 0o700)  # 所有者のみアクセス可

    def save_csv(self, df, filename, include_sensitive=False):
        """
        CSVファイルとして保存

        Parameters:
        -----------
        df : pandas.DataFrame
            保存するデータ
        filename : str
            ファイル名
        include_sensitive : bool
            機密情報を含むかどうか
        """
        # ファイルパスを構築
        filepath = self.output_dir / filename

        # 機密情報を含む場合は警告
        if include_sensitive:
            print("⚠️ 機密情報を含むデータを保存します")
            print(f"   ファイル: {filepath}")
            print("   適切にアクセス制限を設定してください")

        # 保存
        df.to_csv(filepath, index=False, encoding='utf-8-sig')

        # パーミッションを設定（Unixシステムのみ）
        if os.name != 'nt':  # Windows以外
            os.chmod(filepath, 0o600)  # 所有者のみ読み書き可

        print(f"✅ データを保存しました: {filepath}")

        return filepath

    def save_encrypted(self, df, filename, password):
        """
        暗号化して保存（簡易版）

        Parameters:
        -----------
        df : pandas.DataFrame
            保存するデータ
        filename : str
            ファイル名
        password : str
            パスワード

        Note:
        -----
        実際の本番環境では、より強力な暗号化ライブラリ
        （cryptography等）を使用することを推奨
        """
        print("⚠️ この実装は教育目的です")
        print("   本番環境ではより強力な暗号化を使用してください")

        # ここでは暗号化の詳細は省略
        # 実際には cryptography ライブラリなどを使用

        filepath = self.output_dir / filename
        df.to_csv(filepath, index=False, encoding='utf-8-sig')

        print(f"✅ データを保存しました: {filepath}")


# 使用例
saver = SecureDataSaver(output_dir="./secure_output")

# サンプルデータ
df = pd.DataFrame({
    'id': [1, 2, 3],
    'value': [100, 200, 300]
})

# 保存
saver.save_csv(df, "data.csv", include_sensitive=False)
```

---

## 6.5 本番環境での運用

### 6.5.1 ロギングの実装

```python
"""
ロギングの実装
"""
import logging
from datetime import datetime
import os
from pathlib import Path

class DataFetchLogger:
    """
    データ取得のロギングクラス
    """

    def __init__(self, log_dir="./logs"):
        """
        初期化

        Parameters:
        -----------
        log_dir : str
            ログディレクトリ
        """
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(parents=True, exist_ok=True)

        # ロガーを設定
        self.logger = logging.getLogger('JPyDataReader')
        self.logger.setLevel(logging.DEBUG)

        # すでにハンドラが設定されていたらスキップ
        if not self.logger.handlers:
            # ファイルハンドラ
            log_file = self.log_dir / f"data_fetch_{datetime.now():%Y%m%d}.log"
            file_handler = logging.FileHandler(log_file, encoding='utf-8')
            file_handler.setLevel(logging.DEBUG)

            # コンソールハンドラ
            console_handler = logging.StreamHandler()
            console_handler.setLevel(logging.INFO)

            # フォーマッター
            formatter = logging.Formatter(
                '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
            )
            file_handler.setFormatter(formatter)
            console_handler.setFormatter(formatter)

            # ハンドラを追加
            self.logger.addHandler(file_handler)
            self.logger.addHandler(console_handler)

    def log_fetch_start(self, stats_id, params):
        """データ取得開始をログ"""
        self.logger.info(f"データ取得開始: stats_id={stats_id}, params={params}")

    def log_fetch_success(self, stats_id, row_count, elapsed_time):
        """データ取得成功をログ"""
        self.logger.info(
            f"データ取得成功: stats_id={stats_id}, "
            f"rows={row_count}, elapsed={elapsed_time:.2f}s"
        )

    def log_fetch_error(self, stats_id, error):
        """データ取得エラーをログ"""
        self.logger.error(f"データ取得エラー: stats_id={stats_id}, error={error}")

    def log_cache_hit(self, cache_key):
        """キャッシュヒットをログ"""
        self.logger.debug(f"キャッシュヒット: key={cache_key}")

    def log_cache_miss(self, cache_key):
        """キャッシュミスをログ"""
        self.logger.debug(f"キャッシュミス: key={cache_key}")


# 使用例
logger = DataFetchLogger()

logger.log_fetch_start("0003410379", {"limit": 100})
logger.log_fetch_success("0003410379", 100, 1.5)
logger.log_cache_hit("abc123")
```

---

## 📝 練習問題

### 問題1: エラーハンドリング

**課題**: リトライ機能付きで、3回失敗したらメールで通知する機能を実装してください。

### 問題2: パフォーマンス測定

**課題**: データ取得の所要時間を測定し、パフォーマンスレポートを作成してください。

### 問題3: キャッシュ最適化

**課題**: LRU（Least Recently Used）キャッシュを実装してください。

### 問題4: セキュリティチェック

**課題**: データに個人情報が含まれていないかチェックする関数を作成してください。

### 問題5: 本番運用システム

**課題**: 以下の機能を持つ本番運用可能なシステムを構築してください：
1. エラーハンドリング
2. ロギング
3. キャッシング
4. リトライロジック
5. パフォーマンス監視

---

## 🎯 まとめ

この章では以下を学びました：

✅ 包括的なエラーハンドリング
✅ リトライロジックの実装
✅ バッチ処理によるパフォーマンス最適化
✅ データキャッシュ戦略
✅ セキュリティのベストプラクティス
✅ 本番環境での運用テクニック
✅ ロギングとモニタリング

---

## 🎓 チュートリアル完了おめでとうございます！

全6章を通して、JPy-DataReaderの基礎から応用まで学習しました。

### 学習した内容の復習

**第1章**: 基礎編
- インストールと環境設定
- 基本的な3つの関数の使い方

**第2章**: StatsListReader
- 統計表の高度な検索
- フィルタリングとソート

**第3章**: MetaInfoReader
- メタ情報の理解と活用
- 階層構造の分析

**第4章**: StatsDataReader
- 大量データの効率的な取得
- フィルタリング機能

**第5章**: データ分析の実践
- 時系列分析
- 地域比較分析
- 実践プロジェクト

**第6章**: ベストプラクティス
- エラーハンドリング
- パフォーマンス最適化
- セキュリティ

---

## 🚀 次のステップ

1. **実際のプロジェクトで実践**
   - 自分のデータ分析プロジェクトに応用
   - 学んだテクニックを組み合わせる

2. **コミュニティへの参加**
   - GitHubでの質問・議論
   - 改善提案やバグ報告

3. **さらなる学習**
   - pandas, matplotlib の高度な使い方
   - 統計分析手法
   - 機械学習への応用

---

## 📚 参考リソース

- [JPy-DataReader GitHub](https://github.com/kkawailab/jpy-datareader)
- [e-Stat API 仕様](https://www.e-stat.go.jp/api/api-info/api-spec)
- [pandas Documentation](https://pandas.pydata.org/docs/)
- [Python公式ドキュメント](https://docs.python.org/ja/3/)

---

お疲れ様でした！
