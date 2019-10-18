# Android版サンプルコード - LINBLEとの双方向通信

{docsify-updated}

## LINBLEへのデータ送信 - Write

> 参考: BLEとLINBLEの基本制御フロー: LINBLEとの双方向通信: [LINBLEへのデータ送信 - Write](common/flows/communicate-with-linble#linbleへのデータ送信-write)
> 
> ![](../../out/plantuml/sequence_send_write_request.png)

データ送信を行う場合、[BluetoothGattCharacteristic.setValue()]( https://developer.android.com/reference/android/bluetooth/BluetoothGattCharacteristic.html#setValue(byte[]) )で書き込みたいデータをセットしてから、[BluetoothGatt.writeCharacteristic()]( https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#writeCharacteristic(android.bluetooth.BluetoothGattCharacteristic) )を呼び出します。

LINBLEへのデータ送信を行うためには`dataToPeripheral`キャラクタリスティックを操作すればいいので、

```kotlin
val dataToPeripheralCharacteristic = linbleUartService.characteristics
    .firstOrNull { 
        it.uuid == Linble.GattUuid.dataToPeripheral
    } ?: return
```

で`BluetoothGattCharacteristic`オブジェクトを取り出しておき、

```kotlin
dataToPeripheralCharacteristic.value = sendDataByteArray

val succeeded = gatt.writeCharacteristic(dataToPeripheralCharacteristic)

if (!succeeded) {
    // TODO: 失敗時の処理
    return
}
```

のように実装します。

### セットしたデータ長がMTUを超えていた場合、超過分のデータは破棄される

Androidでのデータ送信では、[基本制御フロー](common/flows/communicate-with-linble#linbleへのデータ送信-write)で説明した通り、MTUによるデータ分割の実装は必須です。

Android OSにデータ送信を要求すると、その要求結果が[`BluetoothGattCallback.onCharacteristicWrite()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html#onCharacteristicWrite(android.bluetooth.BluetoothGatt,%20android.bluetooth.BluetoothGattCharacteristic,%20int) )で通知されますので、これをトリガーとして次の分割データを送るように繰り返し処理を構築します。




## LINBLEからのデータ受信 - Notification

> 参考: BLEとLINBLEの基本制御フロー: LINBLEとの双方向通信: [LINBLEからのデータ送信 - Notification](common/flows/communicate-with-linble#linbleからのデータ送信-notification)
> 
> ![](../../out/plantuml/sequence_handle_notification.png)

LINBLEからのNotificationを受信すると、[`BluetoothGattCallback.onCharacteristicChanged()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html#onCharacteristicChanged(android.bluetooth.BluetoothGatt,%20android.bluetooth.BluetoothGattCharacteristic) )が発生します。

Notificationで受信したバイト列は、この通知で渡された`BluetoothGattCharacteristic`の`value`にアクセスすれば確認できます。

```kotlin
override fun onCharacteristicChanged(
    gatt: BluetoothGatt?, 
    characteristic: BluetoothGattCharacteristic?) {
    super.onCharacteristicChanged(gatt, characteristic)

    val notificatedValue = characteristic?.value ?: return

    // notificatedValue ... Notificationで受信したバイト列
}
```
