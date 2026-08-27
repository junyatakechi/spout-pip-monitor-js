# spout-pip-monitor-local-js

SpoutCam などの DirectShow 仮想カメラの映像を Chrome で表示し、Picture-in-Picture で常に最前面に表示するための1ファイルのページです。

Windows には仮想カメラを最前面の小窓で確認する手段が意外と少なく、Microsoft Store の PiP 系アプリはほとんどが UWP アプリのため DirectShow 仮想カメラを列挙できません。Chrome の Picture-in-Picture を使えばこれを回避できます。

## 動作要件

- Windows
- Chrome（または Chromium 系ブラウザ）
- DirectShow で登録された仮想カメラ（[SpoutCam](https://github.com/leadedge/SpoutCam) など）

ビルド、依存パッケージ、サーバーはいずれも不要です。

## 使い方

1. このリポジトリを clone するか、`index.html` をダウンロードする
2. `index.html` を Chrome で開く
3. カメラの使用を許可する
4. プルダウンから仮想カメラ（例: `SPOUTCAM`）を選ぶ
5. 鏡として使う場合は「左右反転」を押す
6. 「PiPで表示」を押す

PiP ウィンドウは常に最前面に表示され、ドラッグで移動、ドラッグで拡大縮小できます。

### file:// で開く場合の制限

`file://` のままでも映像は表示できます。`file:` スキームは [Secure Contexts 仕様](https://w3c.github.io/webappsec-secure-contexts/) で "Potentially Trustworthy" と定められているため、`getUserMedia()` 自体は利用できます。

ただし **カメラの切り替えはできません**。`file://` はオリジンが `null` になるため、Chrome は getUserMedia が成功した後も `enumerateDevices()` を制限し続け、`deviceId` と `label` を空文字で返します（[Chromium 開発陣の説明](https://groups.google.com/g/discuss-webrtc/c/qGdSM-Cs3bc/m/1V0iqPXVCwAJ)）。既定のカメラだけで用が足りるなら `file://` で問題ありません。

切り替えたい場合は、オリジンを持つ場所から開いてください。カメラの許可も記憶されるようになります。

```
python -m http.server 8777 --bind 127.0.0.1
```

GitHub Pages などに置いて `https://` で開く方法でも同じく解決します。

## 仕組み

`navigator.mediaDevices.enumerateDevices()` でビデオ入力を列挙し、選択されたデバイスを `getUserMedia()` で取得します。取得した映像は canvas に描き直し、`canvas.captureStream()` を `<video>` に流して `requestPictureInPicture()` を呼びます。ライブラリ非依存の80行ほどのコードです。

### なぜ canvas を挟むのか

左右反転を CSS の `transform: scaleX(-1)` で行うと、ページ上では反転しますが **PiP ウィンドウには反映されません**。[Picture-in-Picture 仕様](https://www.w3.org/TR/picture-in-picture/)は video 要素に適用されたスタイルを PiP ウィンドウに適用してはならないと定めており、Chromium はこれに従っています（Firefox は `scaleX(-1)` に限り適用するため、挙動が分かれます）。

鏡として使う以上、反転が効くべきなのは PiP ウィンドウの側です。そのため canvas 側で `ctx.setTransform()` によりフレーム自体を反転させ、反転済みの映像を PiP に渡しています。ページ上のプレビューと PiP ウィンドウが常に同じ絵になるという利点もあります。

## なぜ Chrome なのか

Windows のカメラには DirectShow と Media Foundation という2つの経路があります。Media Foundation にはカスタム映像ソースを差し込む仕組みがないため、DirectShow で登録される SpoutCam は Media Foundation 側からは列挙されません。

| アプリ | 経路 | SpoutCam |
| --- | --- | --- |
| Chrome / Discord | Media Foundation →DirectShow にフォールバック | 見える |
| VLC / OBS | DirectShow | 見える |
| Microsoft Store の PiP アプリ・Windows 標準「カメラ」 | Media Foundation | **見えない** |

Chromium は Media Foundation で扱えないデバイスを DirectShow にフォールバックして列挙するため、Chrome からは仮想カメラが見えます。

## うまくいかないとき

**プルダウンに仮想カメラが出てこない**

Chrome を完全に終了してから、DirectShow バックエンドを明示して起動してください。タスクトレイの常駐も含めて終了しないと、既存プロセスに合流してフラグが効きません。

```
chrome.exe --disable-features=MediaFoundationVideoCapture
```

これは Chrome 全体のカメラ処理に影響するため、他のビデオ会議などに不都合が出ないかは確認してください。

**プルダウンの名前が空欄になる**

カメラの使用を許可するまでデバイス名は取得できない仕様です。許可すると名前が表示されます。

## ライセンス

MIT
