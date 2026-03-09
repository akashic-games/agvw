# agvw

agvw は、Web ブラウザで Akashic コンテンツを配置・実行するためのモジュールです。

## 導入

agvw は、 Web ブラウザの script 要素で読み込み可能な .js ファイルとして提供されています。
npm モジュールとして公開されています。

```sh
npm install @akashic/agvw
```

node\_modules/@akashic/agvw/dist/ 以下に上の .js ファイルが置かれるので、必要なファイルを抜き出して利用してください。

特に index.js を script 要素で読み込むと、グローバルに関数 `require()` が定義されます。
`require("@akashic/akashic-gameview-web")` の戻り値から、agvw の API を利用できます。

## 利用例

agvw を使って、`akashic serve` でホストされているコンテンツを実行する HTML ファイルは次のようになります。

```html
<!doctype html>
<title>agvw test</title>
<script type="text/javascript" src="./agvw.js"></script> <!-- node_modules/@akashic/agvw/dist/index.js をリネームしたもの -->
<script>
window.addEventListener("load", () => {
  const agv = require("@akashic/akashic-gameview-web");
  const gameview = new agv.AkashicGameView({
    container: document.getElementById("container"),
    width: 640, // 最大 (かつデフォルト) のゲーム画面幅
    height: 360, // 最大 (かつデフォルト) のゲーム画面高さ
  });

  // content.json (ここでは akashic serve が実行中に提供するもの) を指定してゲームコンテンツを作成。
  const gameContent = new agv.GameContent({
    contentUrl: "http://localhost:3300/contents/0/content.raw.json",
    player: {
      id: "user1",
    },
    playConfig: {
      playId: "dummy_play_id",
      executionMode: agv.ExecutionMode.Active
    }
  });

  // エラーハンドラを設定
  gameContent.addErrorListener({
    onError: function (e) { console.log(e, e.cause); }
  });

  // GameView にコンテンツを配置 (して実行開始)
  gameview.addContent(gameContent);
});
</script>

<div id="container"></div>
```

### content.json

実行する Akashic コンテンツの指定には content.json (の URL) を使用します。
contnet.json は次のような内容のファイルです。

```json
{
    "engine_configuration_version": "3.12.6-1-canvas",
    "engine_urls": [
        "https://.../3.12.6-1-canvas/engineFilesV3_12_6_Canvas.js",
        "https://.../3.12.6-1-canvas/playlogClientV7_3_0.js"
    ],
    "content_url": "https://.../game.json",
    "external": [],
    "content_id": "..."
}
```

|プロパティ名|内容|
|:---|:---|
|`enigne_configuration_version`|コンテンツを実行する akashic-runtime のバージョン|
|`engine_urls`|akashic-runtime のクライアントサイドエンジンコード URL|
|`content_url`|実行するコンテンツの game.json の URL|
|`external`|実行するコンテンツが必要とする外部プラグイン名の配列|
|`content_id`|同一サービス内でコンテンツを識別できるユニークな文字列|

`engine_configuration_version` と `engine_urls` は、実行環境が提供するエンジンのバージョンと URL である必要があります。(TODO: サーバ側動作との連携・具体的なファイルへのリンク)

`external` は、そのコンテンツが必要とする外部プラグイン (`g.game.external.~`) の名前の配列です。
詳細は下記「外部プラグイン」の節を参照してください。

content.json の具体的な例としては、 `akashic serve` を参照してください。
上のコード例でも利用しているとおり、 `akashic serve` は実行しているコンテンツの content.json を `http://localhost:3300/contents/0/content.raw.json` で提供します。

## API 概要

### AkashicGameViewParameterObject インターフェース

`AkashicGameView` (以下 GameView) のコンストラクタ引数の型です。
型のため .js ファイルに実体はありません。

|名前|説明|
|-------|---|
|`container`|コンテンツを表示する `HTMLELement`|
|`width`|GameViewの幅|
|`height`|GameViewの高さ|

### AkashicGameView クラス

AkashicGameView はコンテンツをレイアウトするための領域です。

|名前|説明|
|-------|---|
|`new(AkashicGameViewParameterObject): AkashicGameView`|(コンストラクタ) コンテンツの表示領域を確保して初期化する。|
|`addContent(Content): void`|コンテンツを追加し、ゲームの実行を開始する。|
|`removeContent(Content): void`|コンテンツを削除する。|
|`removeAllContents(): void`|すべてのコンテンツを削除する。|
|`setViewSize(width: number, height: number): void`|表示領域の大きさを変更する。|
|`getViewSize(): {width: number, height: number}`|表示領域の大きさを取得する。|
|`registerExternalPlugin(ExternalPlugin): void`|ゲームが利用可能なプラグインを登録する。|
|`getGameViewSharedObject(): GameViewSharedObject`|GameViewの共有情報を取得する。|
|`destroy(): void`|GameViewを破棄する。(GameView生成時にSharedObjectを省略した場合、destroyと同時にsharedObjectも破棄される点に注意してください。)|
|`destroyed(): boolean`|GameViewが破棄されているかを取得する。|

### GameConfig インターフェース

`GameContent` のコンストラクタ引数の型です。
型のため .js ファイルに実体はありません。

|名前|説明|
|:---|:---|
|`contentUrl`|content.jsonのURL|
|`player`|プレーヤー情報|
|`playConfig`|プレイ情報 (プレイID, 実行モード, プレイログサーバURL, プレイトークン)|
|`contentArea`|(省略可) ゲームの表示領域。省略すると領域全体の大きさ|
|`initialEvents`|(省略可) 起動時に差し込むイベント|
|`audioPdiHandlers`|(省略可) オーディオ処理を差し替えるための場合のハンドラ|

このうち `player` は次の要素を持つ型です。

|名前|説明|
|:---|:---|
|`id`|このコンテンツを実行するプレイヤーのID (`g.game.selfId` に反映)|

`playConfig` はマルチプレイの通信に必要な情報で、次の要素を持つ型です。

|名前|説明|
|:---|:---|
|`playId`|ゲームプレイを識別する値 (`g.game.playId` に反映)|
|`executionMode`|実行モード (後述)|
|`playlogServerUrl`|マルチプレイの通信路サーバの URL|
|`playToken`|マルチプレイの通信路サーバに引き渡すトークン|
|`replayData`|リプレイ情報|
|`replayTargetTime`|リプレイ再生時、再生時刻 (ゲーム開始からの経過ミリ秒) を得るために呼び出されるコールバック関数|

`executionMode` は `agv.ExecutionMode.Active`, `agv.ExecutionMode.Passive`, `agv.ExecutionMode.Replay` のいずれかです。
通信が必要ない (スタンドアロンで実行する) 場合は `Active` を、Akashic System を使ったマルチプレイの場合は `Passive` を、
また Akashic System を使って過去のプレイの再現 (リプレイ) を視聴する場合は `Replay` を指定します。

`Active` の場合、 `palyId`, `executionMode` のみ必要です。
`replayData`, `replayTargetTime` は `Replay` の場合にのみ必要です。

#### 例 (スタンドアロンでコンテンツを実行)

```js
{
  contentUrl: "http://localhost:3300/contents/0/content.raw.json",
  player: {id: "user1"},
  playConfig: {playId: "dummy", executionMode: agv.ExecutionMode.Active}
}
```

#### 例 (マルチプレイの 1 プレイヤーとして実行)

設定値は、適切な通信路を提供するサーバから受け取ったものを与えてください。(TODO: 通信路実装との連携詳細へのリンク)

```js
{
  contentUrl: "http://.../",
  player: {id: "..."},
  playConfig: {
    playId: "...",
    executionMode: agv.ExecutionMode.Passive,
    playlogServerUrl: "ws://.../",
    playToken: "..."
  }
}
```

#### 例 (プレイログを与えてリプレイ再生)

`replayData` は `GameContent#dump()` で得られた値とします。

```js
{
  contentUrl: "http://.../",
  player: {id: "..."},
  playConfig: {
    playId: "...",
    executionMode: agv.ExecutionMode.Replay,
    replayTargetTimeFunc: someClockFunc,  // リプレイ再生の現在時刻を返す関数(プレイ開始からのミリ秒)
    replayData: replayData
  }
}
```

### GameContent クラス

GameContent は Akashic コンテンツの実行単位です。

|名前|説明|
|-------|---|
|`new(GameConfig): GameContent`|(コンストラクタ) ゲームを作成する。実行は GameView に配置したときに開始する。|
|`addErrorListener(ErrorListener)`|発生したエラーを通知するリスナを指定する。|
|`addContentLoadListener(ContentLoadListener)`|コンテンツの実行開始を通知するリスナを指定する。|
|`setContentArea(ContentArea)`|コンテンツの表示領域を設定する。|
|`getContentArea()`|コンテンツの表示領域を取得する。|
|`hide()`|非表示にする。|
|`show()`|表示する。|
|`getGame()`|akashic-engine のインスタンスを取得する。|
|`sendEvents(events: any[])`|ゲームにイベントを送信する。|
|`dumpPlaylog()`|スタンドアロン実行の場合、プレイログをダンプする。|
|`replay(DumpedPlaylog)`|プレイログをもとにプレイを再現する。|
|`getGameContentSize()`|ゲームコンテンツの `game.width`, `game.height` を取得する。|
|`setGameContentSize(GameContenSize)`|ゲームコンテンツの `game.width`, `game.height` を設定する。|
|`getMasterVolume()`|ゲームコンテンツのマスター音量を取得する。|
|`setMasterVolume(volume)`|ゲームコンテンツのマスター音量を設定する。|
|`getExecutionMode()`|ゲームコンテンツの実行モードを取得する。|
|`setExecutionMode(mode)`|ゲームコンテンツの実行モードを設定する。|
|`getReplayTargetTimeFunc()`|リプレイ時の目標時刻を返す関数を取得する。|
|`setReplayTargetTimeFunc(func)`|リプレイ時の目標時刻を返す関数を設定する。|
|`getReplayOriginDate()`|リプレイの基準日時を取得する。|
|`setReplayOriginDate()`|リプレイの基準日時を設定する。|
|`getPlayOriginDate()`|ゲームのプレイ開始日時を取得する。|
|`getReplayOriginDateOffset()`|リプレイデータのプレイログに記録されている開始時刻と目標時刻関数との差を取得する|
|`setTabIndex(tabIndex)`|ゲームコンテンツの要素にtabIndexを付与する。|
|`getTabIndex()`|ゲームコンテンツの要素のtabIndexを取得する。|

## 信頼できないコンテンツについて

agvw は、指定されたコンテンツのスクリプト (.js ファイル) を動的にロードして実行するモジュールです。
ホストしている Web ページ上で任意の動作が可能なため、原則的に **信頼できないコンテンツは実行しないでください** 。

信頼できないコンテンツを実行する場合は、 iframe 要素 (で読み込む独立した HTML) の中でのみ agvw をロードし、親 Web ページからは [`postMessage()`][postmsg] を介して通信・制御するといった防御策が必要です。
少なくとも以下の技術による制限を考慮してください。

* iframe 要素の [sandbox 属性][sandbox-attr] による制限
* [Same Origin Policy (同一生成元ポリシー)][sop] による制限
* [Content Security Poicy][csp] による制限

信頼できないコンテンツの、リソースの取得元、通信先、親フレームへのアクセス、 `document.cookie` や `localStorage` へのアクセスなどは、適切に管理・制限される必要があります。

[postmsg]: https://developer.mozilla.org/ja/docs/Web/API/Window/postMessage
[sop]: https://developer.mozilla.org/ja/docs/Web/Security/Defenses/Same-origin_policy
[csp]: https://developer.mozilla.org/ja/docs/Web/HTTP/Guides/CSP
[sandbox-attr]: https://developer.mozilla.org/ja/docs/Web/HTML/Reference/Elements/iframe#sandbox

## 外部プラグイン

`g.game.external` 以下の値 (外部プラグイン) を必要とするコンテンツを実行する場合は、
事前に `gameview.registerExternalPlugin()` で外部プラグインを登録しておく必要があります。

`print()` メソッドを持つ外部プラグイン `foo` を登録するコードは次のようなものです。

```js
gameview.registerExternalPlugin({
  name: "foo",
  onload: (game) => {
    game.external["foo"] = {
      print: (msg) => {
        console.log(`g.game.external.foo.print: ${msg}`);
      }
    };
  }
});
```

この後、 content.json の `external` に `["foo"]` が指定されたコンテンツを実行すると、そのコンテンツ内では `g.game.external.foo.print()` を呼び出すことができます。

`registerExternalPlugin()` の引数では、以下のプロパティが指定できます。

|プロパティ名|内容|
|:---|:---|
|`name`|プラグイン名。 content.json の `external` で指定されたプラグインを探す際に用いられる。|
|`onload`|ゲームの初期化時に呼び出される関数。引数として `g.game` の値が渡される。この中で `g.game.external[name]` に任意の値を代入することが期待される。|
|`requires`| 依存プラグイン名の配列。省略可。指定すると、このプラグインの `onload` に先立って、指定された各プラグインの `onload` が呼ばれる。|

`requires` で与えられるプラグイン名はそれぞれ `registerExternalPlugin()` されていなければならない点に注意してください。

### 同梱プラグイン

同梱の plugin-instance-storage.js, plugin-instance-storage-limited.js は、外部プラグイン `instanceStorage`, `instanecStorageLimited` を提供します。
本体同様、script 要素で読み込み可能な .js ファイルとして与えられています。
拡張ライブラリ `@akashic-extension/instance-storage` を用いる Akashic コンテンツを実行する場合、この二つのプラグインが必要です。

以下のように `registerExternalPlugin()` してください。

```js
const agv = require("@akashic/akashic-gameview-web");
const isp = require("@akashic/akashic-gameview-web/lib/plugin/InstanceStoragePlugin");
const islp = require("@akashic/akashic-gameview-web/lib/plugin/InstanceStorageLimitedPlugin");

const gameview = new agv.AkashicGameView({ ... });
gameview.registerExternalPlugin(new isp.InstanceStoragePlugin({ storage: window.localStorage }));
gameview.registerExternalPlugin(new islp.InstanceStorageLimitedPlugin());
```

なお信頼できないコンテンツが存在する場合、これらをそのまま利用することは原則できない点に注意してください。(`localStorage` アクセスを許すことが考えにくいため)

信頼できないコンテンツを想定して iframe 内で agvw を扱う場合、外部プラグインとしては「`postMessage()` で親フレームに処理を依頼する中継役」を登録して、実際の処理は親フレーム側で (値の妥当性を検証して) 行うなどの対策が必要です。

## ライセンス

本リポジトリは MIT License の元で公開されています。
詳しくは [LICENSE](./LICENSE) をご覧ください。
