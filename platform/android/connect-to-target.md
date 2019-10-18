# Android版サンプルコード - BLE接続

{docsify-updated}

> 参考: BLEとLINBLEの基本制御フロー: [BLE接続](common/flows/connect-to-target.md)
> 
> ![](../../out/plantuml/sequence_connection.png)

[アドバタイズのスキャン]( platform/android/scan-advertisements )で取得した`ScanRecord`に対し、[`getDevice()`]( https://developer.android.com/reference/android/bluetooth/le/ScanResult.html#getDevice() )を呼ぶことで、[`BluetoothDevice`]( https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html )オブジェクトが得られます。
そして、[`BluetoothDevice.connectGatt()`]( https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html#connectGatt(android.content.Context,%20boolean,%20android.bluetooth.BluetoothGattCallback) )を呼び出すことで、BLE接続を行うことができます。

この接続結果は、[`BluetoothGattCallback.onConnectionStateChange()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html#onConnectionStateChange(android.bluetooth.BluetoothGatt,%20int,%20int) )で通知されます。



## AndroidでのBLE接続は失敗しやすい

Android端末にもよりますが、**`BluetoothGatt.connectGatt()`は比較的高い頻度で失敗します**。

何回かリトライを行えば何事もなかったかのように正常に接続できることが常ですので、リトライ用のロジックは必ず搭載すべきです。

サンプルコードでは接続できるまで無限にリトライをしかけていますが、実際のアプリでは数回失敗したら接続を諦め、ユーザーへエラーを報告した方がよいでしょう。

!> `onConnectionStateChange()`では接続失敗に関するエラーコードも回収できます。しかし、残念ながらほとんどのエラーが`133`という汎用エラーコードで通知されるようになっており、あまり参考にはなりません。


## BluetoothGattCallbackは後から差し替えできない

`BluetoothGattCallback`は、今後に続く様々なBLE制御の処理結果をアプリ側へ通知するために、Android BLEフレームワーク側で使用され続けます。

一方、BLE制御というものは[基本制御フロー]( common/flows/introduction.md )で説明してきた通り、「接続」「GATT準備」「双方向通信」のような複数の制御のステップから成り立ちます。1つのイベントハンドラを複数のステップで横断的に使用することになるため、アプリ開発が進むにつれて、コードの見通しがすぐに悪くなってしまいます。

サンプルコードではこの問題を解消するため、`BluetoothGattCallback`をObserverパターンのように使用できる`BluetoothGattCallbackBridger`クラスを新設して使っています。これにより各ステップでは、各ステップごとに関心のある`BluetoothGattCallback`内イベントを購読管理できるようになっており、見通しの良さが確保されています。
