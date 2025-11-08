# JPy-DataReader チュートリアル - Jupyter Notebook版

## 📓 概要

このディレクトリには、JPy-DataReaderチュートリアルのJupyter Notebook版が含まれています。実際にコードを実行しながら学習できます。

## 📚 Notebook一覧

| Notebook | 章 | 内容 | 難易度 |
|----------|---|------|--------|
| [chapter01_basics.ipynb](./chapter01_basics.ipynb) | 第1章 | 基礎編 - 環境設定と基本操作 | ⭐ 初心者 |
| [chapter02_statslist.ipynb](./chapter02_statslist.ipynb) | 第2章 | StatsListReaderで統計表を探す | ⭐⭐ 中級 |
| [chapter03_metainfo.ipynb](./chapter03_metainfo.ipynb) | 第3章 | MetaInfoReaderでメタ情報を取得 | ⭐⭐ 中級 |
| [chapter04_statsdata.ipynb](./chapter04_statsdata.ipynb) | 第4章 | StatsDataReaderでデータを取得 | ⭐⭐ 中級 |
| [chapter05_advanced_analysis.ipynb](./chapter05_advanced_analysis.ipynb) | 第5章 | データ分析の実践 | ⭐⭐⭐ 上級 |
| [chapter06_best_practices.ipynb](./chapter06_best_practices.ipynb) | 第6章 | ベストプラクティス | ⭐⭐⭐ 上級 |

## 🚀 使い方

### 1. 必要な環境

```bash
# Python 3.12以上
python --version

# Jupyter Notebookのインストール
pip install jupyter

# JPy-DataReaderのインストール
pip install jpy-datareader

# その他の依存パッケージ
pip install python-dotenv pandas matplotlib seaborn
```

### 2. APIキーの設定

プロジェクトルートに `.env` ファイルを作成：

```bash
ESTAT_APP_ID=your_app_id_here
```

APIキーの取得方法は[第1章](./chapter01_basics.ipynb)を参照してください。

### 3. Jupyter Notebookの起動

```bash
# notebooksディレクトリに移動
cd notebooks

# Jupyter Notebookを起動
jupyter notebook
```

ブラウザが開き、Notebookの一覧が表示されます。

### 4. 学習の進め方

1. **第1章から順番に実行**: 各セルを上から順に実行
2. **コードを編集して試す**: 自分でパラメータを変更してみる
3. **練習問題に挑戦**: 各章の練習問題を解いてみる
4. **エラーを恐れない**: エラーが出たら原因を調べて理解を深める

## 💡 Jupyter Notebookの基本操作

### セルの実行

- **Shift + Enter**: セルを実行して次のセルに移動
- **Ctrl + Enter**: セルを実行（移動しない）
- **Alt + Enter**: セルを実行して下に新しいセルを作成

### セルの種類

- **Codeセル**: Pythonコードを書いて実行
- **Markdownセル**: テキストや説明を記述

### よく使うショートカット

- **A**: 上に新しいセルを追加
- **B**: 下に新しいセルを追加
- **D, D**: セルを削除
- **M**: Markdownセルに変換
- **Y**: Codeセルに変換
- **H**: ヘルプ（全ショートカット表示）

## 🎯 学習のポイント

### 初心者の方

1. **焦らず一つずつ**: 各セルをゆっくり実行して理解
2. **エラーメッセージを読む**: エラーから学ぶことが多い
3. **メモを取る**: Markdownセルに自分のメモを追加
4. **練習問題を必ずやる**: 実際に手を動かすことが重要

### 中級者の方

1. **コードを改造してみる**: パラメータや条件を変更
2. **自分のデータで試す**: 興味のある統計データを探す
3. **効率化を考える**: より簡潔なコードを書いてみる

### 上級者の方

1. **応用課題に挑戦**: 複数の技術を組み合わせる
2. **パフォーマンスを測定**: 処理時間を計測して最適化
3. **自分のプロジェクトに適用**: 実際の問題解決に使う

## 📝 よくある質問

### Q1: セルの実行順序は重要ですか？

**A**: はい。特に変数の定義や環境設定のセルは最初に実行する必要があります。

### Q2: エラーが出て進めません

**A**: 以下を確認してください：
- APIキーが正しく設定されているか
- 必要なライブラリがインストールされているか
- セルを上から順に実行しているか
- インターネット接続があるか

### Q3: Notebookを保存するには？

**A**: `Ctrl + S` または メニューの「File」→「Save and Checkpoint」

### Q4: カーネルがクラッシュした場合

**A**: メニューの「Kernel」→「Restart」でカーネルを再起動し、最初から実行し直してください。

### Q5: 練習問題の解答はどこに？

**A**: Notebookには解答は含まれていません。マークダウン版チュートリアルの各章に模範解答があります。

## 🔧 トラブルシューティング

### ModuleNotFoundError が出る

```bash
# 必要なパッケージをインストール
pip install jpy-datareader python-dotenv pandas matplotlib
```

### APIキーエラー

```python
# .envファイルが正しい場所にあるか確認
import os
from dotenv import load_dotenv

load_dotenv()
print(os.getenv('ESTAT_APP_ID'))  # Noneでなければ正常
```

### データ取得エラー

- インターネット接続を確認
- APIキーが有効か確認
- 統計表IDが正しいか確認

## 📚 参考リソース

- [Jupyter Notebook 公式ドキュメント](https://jupyter-notebook.readthedocs.io/)
- [JPy-DataReader 公式リポジトリ](https://github.com/kkawailab/jpy-datareader)
- [e-Stat API 仕様書](https://www.e-stat.go.jp/api/api-info/api-spec)
- [マークダウン版チュートリアル](../)

## 💻 JupyterLab を使う場合

JupyterLabはJupyter Notebookの進化版です：

```bash
# JupyterLabのインストール
pip install jupyterlab

# JupyterLabの起動
jupyter lab
```

## 🎓 次のステップ

全6章を完了したら：

1. **自分のプロジェクトを作成**: 興味のあるデータで分析
2. **GitHub Gistで共有**: 作成したNotebookを共有
3. **コミュニティに貢献**: 改善案やバグ報告

---

## ⚠️ 注意事項

- **APIの利用規約を守る**: e-Stat APIの利用規約を確認
- **大量データ取得に注意**: サーバーに負荷をかけないよう適切なlimitを設定
- **個人情報の取り扱い**: APIキーをNotebookにハードコードしない

---

お疲れ様でした！それでは [第1章](./chapter01_basics.ipynb) から始めましょう！
