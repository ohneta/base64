# base64エンコーダ/デコーダ

ただのBase64のエンコーダ/デコーダ。

## 利用方法
上部のtextareaにエンコードしたい文字列をペーストして [↓endoed]ボタンをクリックするとBase64にエンコードした文字列が下部のtextareaに表示されます。
同様に、下部のtextareaにデコードしたいbase64文字列をペーストして [↑decode]ボタンをクリックするとBase64からデコードした文字列が上部のtextareaに表示されます。
encode textareaにはテキストファイルと画像ファイル(JPEG, PNG)がドロップでき、エンコードできます。
decode textareaにはテキストファイルがドロップできます。
また、 [use dataURL]チェックボックスでdataURLの有/無に対応します。

## サンプルページ
https://service.moyurani.com/base64/


## 開発環境
npm を使用して依存関係をインストールしたら開発環境サーバを起動します。

```bash
npm run dev

# webブラウザと同時に実行する場合
npm run dev -- --open
```

<!-- 
## ビルド
実行するにはnpmを使ってビルドします。

buildディレクトリにできたファイルをすべてwebサーバのドキュメントルートに設置すると動作します。

```bash
npm run build

# ビルド結果をプレビュー
npm run preview -- --open
```
-->