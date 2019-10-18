# Android版サンプルコード - その他特記事項

{docsify-updated}

## 全般

### GATT操作APIではタイムアウト管理を行うべき

全てのGATT操作APIを行う前に[Timer]( https://developer.android.com/reference/java/util/Timer )を起動し、一定時間`BluetoothGattCallback`での応答がなかったらタイムアウトエラーとして処理すべきです。

サンプルコードでは毎回以下のようなコードスニペットでタイムアウト管理をしています。

```kotlin
// コールバック対応の登録
bluetoothGattCallbackBridger.register(object : BluetoothGattCallback() {
    override fun onSomeEvent(gatt: BluetoothGatt?) {
        gattOperationTimeoutDetector?.cancel()  // タイマー解除
        bluetoothGattCallbackBridger.unregister(this)   // コールバック対応の解除

        // ...
    }
})

gattOperationTimeoutDetector = Timer().schedule(gattOperationTimeoutMillis) {
    // TODO: タイムアウトエラー処理
}

val succeeded = gatt.executeSomeOperation()
if (!succeeded) {
    gattOperationTimeoutDetector?.cancel()  // タイマー解除

    // TODO: GATT操作失敗時の処理
}
```


### BLE接続後、ユーザーによってBluetooth機能がオフにされた場合、OSからのエラー通知が一切発生しない

特殊なケースですが、通信中にユーザーによってBluetooth機能がオフにされた場合、LINBLE側は正常に切断されますが、`BluetoothGattCallback.onConnectionStateChange()`は発生しないという挙動になります。

このため、[端末の状態の確認]( platform/android/watch-bluetooth-service-state )で紹介したBluetooth状態の監視機構の搭載が重要となります。

サンプルコードではBluetooth機能がオフにされたら、通信中のBLEオブジェクトを破棄するように実装しています。



## アドバタイズのスキャン

### スキャンを30分間行い続けていると、日和見スキャンモードに格下げされる

[`BluetoothLeScanner`]( https://developer.android.com/reference/android/bluetooth/le/BluetoothLeScanner )が日和見スキャンモード（[`ScanSettings.SCAN_MODE_OPPORTUNISTIC`]( https://developer.android.com/reference/android/bluetooth/le/ScanSettings.html#SCAN_MODE_OPPORTUNISTIC) ）に格下げされると、そのスキャナーへのアドバタイズ発見通知は基本的に発生しなくなり、他アプリがスキャンをした際についでに通知してくれる、という奇妙な動作状態になります。

長時間スキャンを行う必要があるアプリは、スキャン開始後、例えば5分に1回など、定期的にスキャン処理を再起動するようにしてください。



### スキャン開始を30秒以内に5回呼び出すとエラーになる

Android7より、スキャン開始・スキャン停止を30秒以内に5回呼び出すと、そのスキャン開始が内部で失敗するようになっています。
LogCatには以下のエラーログが記録されます（`Show only selected application`フィルタが有効になっていると表示されません。`No Filters`を選択してください）。

```
App 'xxx' is scanning too frequently
```

**厄介なことに、この時、[`ScanCallback.onScanFailed()`]( https://developer.android.com/reference/android/bluetooth/le/ScanCallback.html#onScanFailed(int) )によるスキャンエラー報告は発生してくれません**。したがって、ユーザーに対してスキャン開始ができなかったことを知らせることもできません。

このエラーは、意図的なスキャン開始・スキャン停止を繰り返すコードを実装してなくても、不意に遭遇してしまうことがあります。
例えば、画面遷移とスキャンの開始・停止を連動させている場合、ユーザーが画面を行ったり来たりするだけでこのエラー発生条件が満たされてしまいます。

?> サンプルコードでは、この問題に対する対策コードは搭載していません。対応策としては、スキャン開始前に「5回前のスキャン開始時刻」と「現在の時刻」の時間差を確認し、これが30秒以内だったらその差だけスキャン開始を遅延させるロジックを追加することが考えられます。

