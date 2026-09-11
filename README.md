# サブスク管理アプリ

サブスクの一覧・毎月/年間の合計金額・おすすめ解約日の計算・カレンダー表示・毎月の支出推移グラフをまとめて管理する、スマホ向けのシンプルなWebアプリです。

## 使い方

`index.html` を開くだけで動きます。ビルド不要、外部ライブラリ不要の単一HTMLファイルです。

- **ローカルで試す**: このファイルをブラウザで直接開くか、`npx serve` などの簡易サーバーで配信してください。
- **公開する**: このリポジトリをGitHub Pagesで公開すれば、スマホのブラウザからいつでもアクセスできます(Settings → Pages → Source を `main` ブランチの `/`(root)に設定)。

## データの保存場所

基本はブラウザの `localStorage` です。サブスクの情報・毎月の記録・アップロードしたロゴ画像は、まずこの端末内に保存されます。

**Googleでログインすると、Firebase(Firestore)にも自動で同期されます。** ログインすれば、別の端末で同じGoogleアカウントでログインしたときに同じデータが見られます。ログインしなければ、これまで通り端末内だけで完結します(サーバーには一切送信されません)。

Firebaseへの同期は **GitHub Pagesで開いたときのみ**動作します。Claude Artifacts上ではFirebaseのSDKがセキュリティ制限で読み込めないため、自動的にローカル保存のみのモードになります。

ロゴ画像は、保存容量を圧迫しないよう自動で128px四方に縮小してから保存しています。

ログインしない/できない場合に備えて、ホーム画面下部に**バックアップを保存/復元する**リンクも用意しています。「バックアップを保存」を押すと、全データを1つのJSONファイルとして端末に保存できます(スマホでは「ダウンロード」フォルダなど、ブラウザの既定の保存先に置かれます)。「復元する」からそのファイルを選べば元に戻せます。

### Firebaseのセットアップ(このリポジトリを自分用に使う場合)

1. [Firebaseコンソール](https://console.firebase.google.com/)でプロジェクトを作成
2. 「Webアプリを追加」して `firebaseConfig` を取得し、`index.html` 内の該当箇所に貼り替える
3. Authentication → Sign-in method → **Google** を有効化
4. Firestore Database を作成(ロケーションは任意、本番環境モード)
5. Firestore の Rules を以下に置き換えて公開する

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

これにより、各ユーザーは自分自身のデータ(`users/{自分のuid}`)にしか読み書きできません。

## 構成

すべて `index.html` 1ファイルの中に、HTML・CSS・JavaScriptがまとまっています。主なセクション:

- 日付計算・請求日/解約目安日のロジック
- ホーム画面(一覧・合計金額)
- 追加/編集画面
- 詳細画面
- カレンダー画面
- 支出推移グラフ画面
