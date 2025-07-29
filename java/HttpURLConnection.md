`HttpURLConnection` は、Java 標準ライブラリ（`java.net` パッケージ）に含まれるクラスで、**HTTP 通信（GET, POST など）を行うための基本的な手段**です。サードパーティのライブラリ（例：Apache HttpClient や OkHttp）を使わない場合に、もっともよく使われる低レベルな通信APIです。

---

## 🔍 概要

```java
java.net.HttpURLConnection
```

* `URL` クラスと組み合わせて使用する
* HTTPリクエスト（GET / POST / PUT / DELETEなど）を送信できる
* Java SE に標準搭載（追加ライブラリ不要）
* Java 11 以降は `HttpClient` の使用が推奨されるが、古いプロジェクトでは現役

---

## 🧱 主な特徴

| 特徴        | 説明                                  |
| --------- | ----------------------------------- |
| プロトコル     | HTTP/HTTPS 両方対応（自動切り替え）             |
| リクエストヘッダー | `setRequestProperty` で追加可能          |
| リクエストボディ  | `setDoOutput(true)` でPOSTなどの送信が可能に  |
| タイムアウト設定  | 接続・読み取りタイムアウトの個別設定が可能               |
| リダイレクト    | 自動追従 or 手動処理可能                      |
| 認証        | `Authorization` ヘッダーで設定（Basic 認証など） |

---

## 🧪 使用例（GET）

```java
URL url = new URL("https://example.com/api/data");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
conn.setConnectTimeout(5000); // 接続タイムアウト
conn.setReadTimeout(5000);    // 読み込みタイムアウト

int status = conn.getResponseCode();
BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
String inputLine;
StringBuilder content = new StringBuilder();
while ((inputLine = in.readLine()) != null) {
    content.append(inputLine);
}
in.close();
conn.disconnect();

System.out.println(content.toString());
```

---

## 📤 使用例（POST）

```java
URL url = new URL("https://example.com/api/login");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("POST");
conn.setDoOutput(true);  // POSTボディ送信のために必要
conn.setRequestProperty("Content-Type", "application/json");

String json = "{\"username\":\"user\", \"password\":\"pass\"}";
try (OutputStream os = conn.getOutputStream()) {
    byte[] input = json.getBytes("utf-8");
    os.write(input, 0, input.length);
}

int status = conn.getResponseCode();
BufferedReader br = new BufferedReader(new InputStreamReader(conn.getInputStream(), "utf-8"));
StringBuilder response = new StringBuilder();
String responseLine;
while ((responseLine = br.readLine()) != null) {
    response.append(responseLine.trim());
}
System.out.println(response.toString());
```

---

## 🛠 よく使うメソッド

| メソッド名                      | 説明             |
| -------------------------- | -------------- |
| `setRequestMethod("GET")`  | リクエスト種別を指定     |
| `setRequestProperty(k, v)` | リクエストヘッダーを追加   |
| `getInputStream()`         | 成功時のレスポンス取得    |
| `getErrorStream()`         | エラー時のレスポンス取得   |
| `getResponseCode()`        | HTTPステータスコード取得 |
| `disconnect()`             | 接続終了（リソース解放）   |

---

## ⚠️ 注意点

| 注意事項      | 内容                                               |
| --------- | ------------------------------------------------ |
| SSL証明書    | HTTPS 通信で自己署名証明書を使う場合、検証エラーになることあり               |
| リソース開放    | 明示的に `disconnect()` や `try-with-resources` を使うべき |
| Java 11以降 | `java.net.http.HttpClient` を推奨（非同期・簡潔）           |
| リクエスト本文   | `setDoOutput(true)` を忘れると `POST` の本文が送信されない      |

---

## ✅ 適しているケース

* ライブラリを使いたくない（標準機能で完結したい）
* 小規模プロジェクト／単純なGET・POST処理だけ必要
* 古い環境（Java 8以前）を使っている

---

## ❌ 適していないケース

* 認証やリダイレクト、Cookie管理などが複雑なAPI連携
* 高度な非同期通信やHTTP2を扱いたい（→ `HttpClient`, `OkHttp`, `Retrofit` など推奨）

---

## 📎 まとめ

| 観点    | 内容                               |
| ----- | -------------------------------- |
| 標準性   | Java標準、追加ライブラリ不要                 |
| 使いやすさ | 低レベルだがシンプル。制御は手動                 |
| 拡張性   | 高くはない（複雑なAPI処理には不向き）             |
| モダン代替 | Java 11以降は `HttpClient` クラスを使用推奨 |

---

必要であれば、Spring の `RestTemplate` や `WebClient`、Java 11の `HttpClient` との比較表も作成できます。
