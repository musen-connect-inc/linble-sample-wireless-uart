# iOS版サンプルコード - その他特記事項

{docsify-updated}

## 全般

### GATT操作APIではタイムアウト管理を行うべき

全てのGATT操作APIを行う前に[Timer](https://developer.apple.com/documentation/foundation/timer)を起動し、一定時間`CBPeripheralDelegate`での応答がなかったらタイムアウトエラーとして処理すべきです。

サンプルコードでは毎回以下のようなコードスニペットでタイムアウト管理をしています。

```swift
// CBPeripheralDelegate呼び出し
func peripheral(_ peripheral: CBPeripheral) {
    timeoutTimer?.invalidate()  // タイマー解除

    // ...
}

timeoutTimer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: false) { _ in
    // TODO: タイムアウトエラー処理
}
```

### BLE接続後、ユーザーによってBluetooth機能がオフにされた場合、OSからのエラー通知が発生しない場合がある

特殊なケースですが、通信中にユーザーによってiOS自体のBluetooth機能がオフにされた場合、LINBLE側は正常に切断されますが、`centralManager(_:didDisconnectPeripheral:error:)`が発生しない場合があります。

設定アプリからBluetooth機能をオフにした場合は発生しますが、コントロールセンターからオフにした場合や、機内モードと連動してオフになった場合は発生しません。

このため、[端末の状態の確認](platform/ios/watch-bluetooth-service-state)で紹介したBluetooth状態の監視機構の搭載が重要となります。

サンプルコードではBluetooth機能がオフにされたら、通信中のBLEオブジェクトを破棄するように実装しています。
