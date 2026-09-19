# Block AI Overviews

Google検索をした際、URLの末尾に `&udm=14` を追加する拡張機能。また、検索クエリ以外の不要なパラメータも削除する。

## 構成

```
hide-overview/
├── manifest.json
└── rules.json
```

## manifest.json

Chromeに対してルールベースでURLの処理を行うようにしている。

| 項目 | 内容 |
|---|---|
| `permissions` | [`declarativeNetRequest`](https://developer.chrome.com/docs/extensions/reference/api/declarativeNetRequest): URLのリダイレクト・書き換えを行うためのAPI権限 |
| `host_permissions` | `*://www.google.com/*`: google.comへのアクセスに対してのみ動作を許可 |
| `declarative_net_request` | 書き換えルールを `rules.json` から読み込む設定 |

## rules.json

実際にどのように書き換えるかというルールを定義している。

### condition

以下の条件を定義している。

```json
"regexFilter": "^(https?://www\\.google\\.com)/search\\?(?:.*&)?q=([^&]+)(?:&.*)?$"
```

| 正規表現 | 意図 |
|---|---|
| `(https?://www\.google\.com)` | 括弧で囲み `\1` として使用 |
| `/search\?` | パスが `/search?` であるか |
| `(?:.*&)?q=([^&]+)` | クエリの途中にある `q=検索ワード` を検出し、検索ワード部分だけを `\2` として使用 |
| `(?:&.*)?$` | その後に続く他のパラメータは許容するが、使用しない |
 
`"resourceTypes": ["main_frame"]` は、アドレスバーへの入力やページ全体の遷移のみを対象とし、画像や広告といったバックグラウンドの通信は対象外となる。

### action

条件に一致したURLを以下の形式に置き換えている。

```json
"regexSubstitution": "\\1/search?q=\\2&udm=14"
```
 
`condition` で取得したドメイン (`\1`) と検索ワード (`\2`) のみを使用し、新しいURLを作成する(以下の例ではドメインを省略している)。

- Before: `/search?q=chrome+extension&hl=ja&sourceid=chrome`
- After: `/search?q=chrome+extension&udm=14`

元のURLに含まれていた `hl` や `sourceid` などの余計なパラメータはすべて削除され、`q` と `udm=14` のみから構成されるURLになる。
