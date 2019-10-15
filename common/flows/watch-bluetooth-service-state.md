# 端末の状態の確認

{docsify-updated}

まずはセントラルアプリ稼動端末のBluetooth機能状態を確認し、[スキャン](common/flows/scan-advertisements.md)が可能な状態になっているかをチェックします。

!> スマホ・PCのBluetooth機能は、ユーザーによって無効化されている可能性があります。この場合、ユーザーにBluetoothを有効化してもらうように案内ダイアログを表示します。

Bluetooth機能の状態がONであることがわかったら、[スキャン](common/flows/scan-advertisements.md)を開始します。

![](../../out/plantuml/sequence_start.png)

?> シーケンス図のフレームの右端への矢印は、各プラットフォームごとのSDKのBluetooth制御APIの呼び出しを意味します。それぞれの矢印で使うべきAPIについての情報は、[Android](https://developer.android.com/guide/topics/connectivity/bluetooth-le.html)、[iOS](https://developer.apple.com/documentation/corebluetooth)、[Windows10-UWP](https://docs.microsoft.com/ja-jp/windows/uwp/devices-sensors/bluetooth-low-energy-overview)ごとに、SDK公式のBLEについての解説ドキュメントを参照してください。
