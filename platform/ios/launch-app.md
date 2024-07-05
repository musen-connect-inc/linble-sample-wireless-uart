# iOS版サンプルコード - アプリの起動

{docsify-updated}

SwiftUIでは、`@main`属性がついたApp構造体がエントリーポイントとなります。

アプリを起動させると、ContentView の [`onAppear`](<https://developer.apple.com/documentation/swiftui/view/onappear(perform:)>) が呼ばれて処理が開始されます。


## 使用許可取得

Core Bluetoothフレームワークを使用する場合は、Info.plistに[`Privacy – Bluetooth Always Usage Description`](https://developer.apple.com/documentation/bundleresources/information_property_list/nsbluetoothalwaysusagedescription)を追加する必要があります。

Core Bluetooth フレームワークを使用する際に、アプリ動作中にユーザーと対話をしてそのAPIの利用許可を得る必要があります。
Info.plistのvalueで設定した値がユーザに対話メッセージとして表示されます。

## BLE制御の開始

Viewの`onAppear`時に`WirelessUartController.start()`がコールされ、BLE処理が開始されます。

以下、[BLEとLINBLEの基本制御フロー](common/flows/introduction.md)で登場していた`BluetoothCentralController`クラスで対応した処理が行われていきます。
