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
5. 「PiPで表示」を押す

PiP ウィンドウは常に最前面に表示され、ドラッグで移動、ドラッグで拡大縮小できます。

`file://` のまま動きます。`file:` スキームは [Secure Contexts 仕様](https://w3c.github.io/webappsec-secure-contexts/) で "Potentially Trustworthy" と定められており、`getUserMedia()` が利用できるためです。ローカルサーバーを立てる必要はありません。

ただし `file://` はホストを持たない特殊なオリジンのため、環境によってはカメラの許可が記憶されず、開くたびに許可ダイアログが出ることがあります。煩わしい場合は任意のローカルサーバーから配信してください。

```
python -m http.server 8777 --bind 127.0.0.1
```

## 仕組み

`navigator.mediaDevices.enumerateDevices()` でビデオ入力を列挙し、選択されたデバイスを `getUserMedia()` で取得して `<video>` に流し、`requestPictureInPicture()` を呼ぶだけです。全体で50行ほどのライブラリ非依存のコードです。

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
