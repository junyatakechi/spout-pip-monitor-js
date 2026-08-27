# spoutcam-pip

SpoutCam などの DirectShow 仮想カメラの映像を Chrome で表示し、Picture-in-Picture で常に最前面に出すためのページ。

## 使い方

`index.html` を Chrome で開き、プルダウンからカメラを選んで「PiPで表示」を押す。

サーバーは不要。`file:` スキームは Secure Contexts 仕様で "Potentially Trustworthy" と定められているため、`file://` のままで `getUserMedia` が使える。

## 背景

Webcam PiP や Windows 標準の「カメラ」アプリは UWP アプリで、カメラの列挙に Media Foundation を使う。
Media Foundation にはカスタム映像ソースを差し込む仕組みがないため、DirectShow で登録される SpoutCam は列挙されない。

Chromium 系は Media Foundation で拾えないデバイスを DirectShow にフォールバックして列挙するので、Chrome からは見える。

| 経路 | SpoutCam |
| --- | --- |
| Chrome / Discord | 見える |
| VLC / OBS | 見える |
| UWP アプリ（Webcam PiP・標準カメラ） | 見えない |

もし Chrome のプルダウンに出てこない場合は、Chrome を完全終了してから DirectShow バックエンドで起動する。

```
chrome.exe --disable-features=MediaFoundationVideoCapture
```
