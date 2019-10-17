# Android版サンプルコードの解説

本資料では、Android版サンプルコードである[linble-sample-wireless-uart-android]( https://github.com/musen-connect-inc/linble-sample-wireless-uart-android )について、その構造や要点について解説しています。

元となる設計情報については[BLEとLINBLEの基本制御フロー](common/flows/introduction.md)を参照してください。


## 前書き

* サンプルコード内のコメントはBLE制御をする上で重要になること・分かりづらいところについてのみ記載しています。

* BLE APIの使い方について紹介するのが本サンプルコードの目的となっています。Androidアプリ自体の基本的な開発方法に関しては解説は省略します。
    * 特に、`Acitivity`まわりには[Android Jetpack]( https://developer.android.com/jetpack?hl=JA )の[ViewModel]( https://developer.android.com/topic/libraries/architecture/viewmodel?hl=JA )・[LiveData]( https://developer.android.com/topic/libraries/architecture/livedata?hl=JA )を使用していることに注意してください。

* 実際に参考にすべきコードは[`app/src/main/java/com.musenconnect.linble.sample.wirelessuart.android/`]( https://github.com/musen-connect-inc/linble-sample-wireless-uart-android/tree/master/app/src/main/java/com/musenconnect/linble/sample/wirelessuart/android )内に配置されています。

### 接続対象BDアドレスの変更

アプリをビルドする前に、[`WirelessUartController`クラス内の`defaultTargetLinbleAddress`の文字列]( https://github.com/musen-connect-inc/linble-sample-wireless-uart-android/blob/master/app/src/main/java/com/musenconnect/linble/sample/wirelessuart/android/common/WirelessUartController.kt#L16 )を適切に変更してください。

```kotlin
companion object {
    val defaultTargetLinbleAddress: String = "FFFFFFFFFFFF" // `BTM` コマンドで確認できるBDアドレス文字列をここに貼り付けてください。
        .insertColons()
}
```

この文字列は接続対象のBDアドレスとして使用されます。
コメントでも書いている通り、`BTM`コマンドによって取得できた文字列をここにセットしてください。



## パッケージ構造

### パッケージ

#### appパッケージ

アプリのエントリポイントとなる`MainActivity`、および、[Runtime Permission制御]( #位置情報についてのruntime-permission取得 )の処理を行うための`RuntimePermissionHandler`クラスが配置されています。

#### commonパッケージ

**サンプルコードの設計**で記述した

* [BLEとLINBLEの基本制御フロー](common/flows/introduction.md)
* [使用するUART通信仕様](common/command-interface.md)
* [全体構造](common/classes.md)

の設計を、**Android BLE APIを使わずに**そのままKotlinでコーディングした抽象層です。

#### concreteパッケージ

[commonパッケージ]( #commonパッケージ )のBLE抽象化部分について、**実際のAndroid BLE APIを使用するようにして**具体的に実装した層です。





## サンプルコードの流れ


### アプリの起動

`MainActivity`が立ち上がり、Android OSから`onCreate()`や`onStart()`がコールされます。

`onStart()`時、`MainActivityViewModel`内の`RuntimePermissionHandler`によって、[Runtime Permission]( #位置情報についてのruntime-permission取得 )に関する確認が行われます。

?> `MainActivityViewModel`は、画面回転時の`Activity`再生成の影響を受けません。ここにBLE制御を管理するオブジェクトを所持させることで、BLE処理が継続するようになっています。逆に言えば、`Activity`にBLE制御担当オブジェクトを所持させてしまうと、画面回転のたびにBLE通信が切断されるような動きになってしまいます。



#### 位置情報についてのRuntime Permission取得

[Runtime Permission]( https://developer.android.com/training/permissions/requesting?hl=ja9 )はAndroid 6で導入された、「アプリ起動後におけるデバイス機能の利用認可の仕組み」です。
Android SDKの一部APIを利用するためには、Google Playからのインストール時点とは別に、アプリ動作中にユーザーと対話をしてそのAPIの利用許可を得る必要があります。

**Androidの思想において、Bluetooth機能は「位置情報」機能のカテゴリの中に入れられており、**Bluetooth系のAPIを利用するためには[`Manifest.permission.ACCESS_FINE_LOCATION`]( https://developer.android.com/reference/android/Manifest.permission#ACCESS_FINE_LOCATION )（Android9以下では[`ACCESS_COARSE_LOCATION`]( https://developer.android.com/reference/android/Manifest.permission#ACCESS_COARSE_LOCATION) ）のRuntime Permission取得が必要です。

?> ユーザーは「位置情報」という機能について、**「端末のGPS情報を記録・外部送信して悪用されるかもしれない」**といったネガティブな印象を持っている可能性があり、位置情報機能を要求してきたアプリに対して不信感を抱くかもしれません。Runtime Permissionを取得する前に、「Bluetooth機能の利用のために位置情報利用権限が必要である」「得られた権限はBLE通信のために利用し、位置の追跡のためには使用しない」ということを説明すべきです。サンプルコードでは、Runtime Permissionの取得用システムダイアログを表示する前に、この点についてユーザーへ説明する[`AlertDialog`]( https://developer.android.com/guide/topics/ui/dialogs?hl=ja )を表示させています。





### BLE制御の開始

Runtime Permissionが取得済みである場合、`WirelessUartController.start()`がコールされ、BLE処理が開始されます。

以下、[BLEとLINBLEの基本制御フロー](common/flows/introduction.md)に対応した処理が行われていきますが、あちらのドキュメントで登場していた`BluetoothCentralController`インタフェースの実体は`ConcreteBluetoothCentralController`クラス、`Linble`インタフェースの実体は`ConcreteLinble`クラスが適用されます。


#### 端末の状態の確認

> 参考: BLEとLINBLEの基本制御フロー: [端末の状態の確認](common/flows/watch-bluetooth-service-state.md)

AndroidでBluetooth状態の監視を行う場合、[ブロードキャスト]( https://developer.android.com/guide/components/broadcasts.html )の仕組みを用います。

##### 位置情報機能の状態確認も必要

ただし、**Android6以降でBLEスキャンを利用するためには、位置情報機能が有効になっている必要があります。**
昨今のAndroid BLEアプリでは、位置情報機能の状態確認についても同時に行われるべきです。

Bluetooth機能の状態が変更されると、`BluetoothAdapter.ACTION_STATE_CHANGED`がブロードキャストされます。位置情報機能の状態が変更されると、`LocationManager.PROVIDERS_CHANGED_ACTION`がブロードキャストされます。

これらのアクションを[`IntentFilter.addAction()`]( https://developer.android.com/reference/android/content/IntentFilter.html#addAction(java.lang.String) )で監視対象として、その`IntentFilter`を使うレシーバを[`Context.registerReceiver()`]( https://developer.android.com/reference/android/content/Context.html#registerReceiver(android.content.BroadcastReceiver,%20android.content.IntentFilter) )で登録します。

```kotlin
applicationContext.registerReceiver(bluetoothStateBroadcastReceiver, IntentFilter().also {
    it.addAction(BluetoothAdapter.ACTION_STATE_CHANGED)
    it.addAction(LocationManager.PROVIDERS_CHANGED_ACTION)
})
```

登録された`bluetoothStateBroadcastReceiver`の[`onReceive()`](https://developer.android.com/reference/android/content/BroadcastReceiver#onReceive(android.content.Context,%20android.content.Intent))が呼び出されたら、その時のBluetooth機能・位置情報機能の状態を確認し直し、それらを統合したものを現在の端末のBluetooth状態として取り扱います。

```kotlin
private fun updateCurrentDeviceBluetoothState(context: Context) {
    val bluetoothIsPoweredOn = BluetoothAdapter.getDefaultAdapter().isEnabled

    currentDeviceBluetoothState = if (bluetoothIsPoweredOn) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            /*
            Android6以降では、位置情報機能が有効になっているかもBLEスキャンの利用可否に繋がります。

            `LocationManager.NETWORK_PROVIDER` が有効になっていれば、BLEスキャンを利用可能と見なすことができます。
            */
            val locationManager = context.getSystemService(LocationManager::class.java)
            if (locationManager.isProviderEnabled(LocationManager.NETWORK_PROVIDER)) {
                DeviceBluetoothState.PoweredOn
            }
            else {
                DeviceBluetoothState.PoweredOnButDisabledLocationService
            }
        }
        else {
            DeviceBluetoothState.PoweredOn
        }
    }
    else {
        DeviceBluetoothState.PoweredOff
    }
}
```

?> [構造図](common/classes.md)内の列挙型`DeviceBluetoothState`は、端末のBluetooth状態として`Unknown`・`PoweredOff`・`PoweredOn`の3値を想定していましたが、Androidのサンプルコードでは新たに`PoweredOnButDisabledLocationService`の値を追加しています。


#### アドバタイズのスキャン

> 参考: BLEとLINBLEの基本制御フロー: [アドバタイズのスキャン](common/flows/scan-advertisements.md)

AndroidでのBLEスキャンには、[`BluetoothLeScanner`]( https://developer.android.com/reference/android/bluetooth/le/BluetoothLeScanner )を使用します。

このオブジェクトは[`BluetoothAdapter.getBluetoothLeScanner()`]( https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#getBluetoothLeScanner() )で取得できます。ただし、Bluetooth状態がオフのときは`null`が返りますので、事前に[端末の状態の確認](#端末の状態の確認)を行っておくことが重要です。

スキャン結果は[`ScanCallback.onScanResult()`]( https://developer.android.com/reference/android/bluetooth/le/ScanCallback.html#onScanResult(int,%20android.bluetooth.le.ScanResult) )で渡される[`ScanResult`]( https://developer.android.com/reference/android/bluetooth/le/ScanResult.html )オブジェクトから取り出すことができます。

なお、スキャン開始時に指定した[`ScanCalback`]( https://developer.android.com/reference/android/bluetooth/le/ScanCallback )オブジェクトはスキャン停止のためにも必要になるので、`ScanCalback`はフィールドで保持しておく必要があります。



#### BLE接続

> 参考: BLEとLINBLEの基本制御フロー: [BLE接続](common/flows/connect-to-target.md)

[アドバタイズのスキャン](#アドバタイズのスキャン)で取得した`ScanRecord`に対し、[`getDevice()`]( https://developer.android.com/reference/android/bluetooth/le/ScanResult.html#getDevice() )を呼ぶことで、[`BluetoothDevice`]( https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html )オブジェクトが得られます。
そして、[`BluetoothDevice.connectGatt()`]( https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html#connectGatt(android.content.Context,%20boolean,%20android.bluetooth.BluetoothGattCallback) )を呼び出すことで、BLE接続を行うことができます。

この接続結果は、[`BluetoothGattCallback.onConnectionStateChange()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html#onConnectionStateChange(android.bluetooth.BluetoothGatt,%20int,%20int) )で通知されます。

##### AndroidでのBLE接続は失敗しやすい

Android端末にもよりますが、**`BluetoothGatt.connectGatt()`は比較的高い頻度で失敗します**。

何回かリトライを行えば何事もなかったかのように正常に接続できることが常ですので、リトライ用のロジックは必ず搭載すべきです。

サンプルコードでは接続できるまで無限にリトライをしかけていますが、実際のアプリでは数回失敗したら接続を諦め、ユーザーへエラーを報告した方がよいでしょう。

!> `onConnectionStateChange()`では接続失敗に関するエラーコードも回収できます。しかし、残念ながらほとんどのエラーが`133`という汎用エラーコードで通知されるようになっており、あまり参考にはなりません。


##### BluetoothGattCallbackは後から差し替えできない

`BluetoothGattCallback`は、今後に続く様々なBLE制御の処理結果をアプリ側へ通知するために、Android BLEフレームワーク側で使用され続けます。

一方、BLE制御というものは[基本制御フロー]( common/flows/introduction.md )で説明してきた通り、「接続」「GATT準備」「双方向通信」のような複数の制御のステップから成り立ちます。1つのイベントハンドラを複数のステップで横断的に使用することになるため、アプリ開発が進むにつれて、コードの見通しがすぐに悪くなってしまいます。

サンプルコードではこの問題を解消するため、`BluetoothGattCallback`をObserverパターンのように使用できる`BluetoothGattCallbackBridger`クラスを新設して使っています。これにより各ステップでは、各ステップごとに関心のある`BluetoothGattCallback`内イベントを購読管理できるようになっており、見通しの良さが確保されています。


#### GATT準備

> 参考: BLEとLINBLEの基本制御フロー: [GATT準備](common/flows/prepare-gatt.md)

##### サービス検索とキャラクタリスティック検索は同時に行われる

サービス検索とキャラクタリスティック検索は、[BluetoothGatt.discoverServices()]( https://developer.android.com/reference/android/bluetooth/BluetoothGatt#discoverServices() )を呼ぶことで同時に行われます。

この操作により、接続しているGATTサーバー側の全てのサービス・キャラクタリスティックが調査されます。

調査が成功したら[`BluetoothGattCallback.onServicesDiscovered()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback#onServicesDiscovered(android.bluetooth.BluetoothGatt,%20int) )が呼び出されます。

##### Notification許可が面倒

AndroidでのNotification許可は、少々多めのコードを書かなければなりません。

1. サービス・キャラクタリスティック検索完了後、Notification許可を行いたいキャラクタリスティックに関する[`BluetoothGattCharacteristic`]( https://developer.android.com/reference/android/bluetooth/BluetoothGattCharacteristic.html )オブジェクトを取り出します。

[LINBLE UART ServiceのUUID定義]( common/flows/prepare-gatt#linble-uart-serviceのuuid・属性定義 )は、

```kotlin
interface Linble {
    object GattUuid {
        val linbleUartService
            = UUID.fromString("27ADC9CA-35EB-465A-9154-B8FF9076F3E8")!!

        val dataFromPeripheral
            = UUID.fromString("27ADC9CB-35EB-465A-9154-B8FF9076F3E8")!!

        val dataToPeripheral
            = UUID.fromString("27ADC9CC-35EB-465A-9154-B8FF9076F3E8")!!
    }
}
```

のように実装できます。

Notification許可をしたいのは`dataFromPeripheral`に関する`BluetoothGattCharacteristic`です。
Kotlinのコレクション操作関数[`firstOrNull()`]( https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/first-or-null.html )を使えば、`dataFromPeripheralCharacteristic`の取り出しは

```kotlin
val linbleUartService = gatt.services
    ?.firstOrNull {
        it.uuid == Linble.GattUuid.linbleUartService
    } ?: return

val dataFromPeripheralCharacteristic = linbleUartService.characteristics
    .firstOrNull { 
        it.uuid == Linble.GattUuid.dataFromPeripheral 
    } ?: return
```

のように書けます。

2. 取り出した`dataFromPeripheralCharacteristic`を使って、[`BluetoothGatt.setCharacteristicNotification()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGatt#setCharacteristicNotification(android.bluetooth.BluetoothGattCharacteristic,%20boolean) )を実行します。

```kotlin
val succeeded = gatt
    .setCharacteristicNotification(dataFromPeripheralCharacteristic, true)

if (!succeeded) {
    // TODO: 失敗時の処理
    return
}
```

これによって、Android OSがこのキャラクタリスティックから発生したNotificationを受け取ったとき、このアプリにイベントを横流ししてくれるようになります。

3. `dataFromPeripheralCharacteristic`から、*Client Characteristic Configuration Descriptor（通称CCCD）*についての[`BluetoothGattDescriptor`]( https://developer.android.com/reference/android/bluetooth/BluetoothGattDescriptor )オブジェクトを取得します。

```kotlin
val dataFromPeripheralCccd = dataFromPeripheralCharacteristic
        .getDescriptor(BluetoothLowEnergySpec.GattUuid.cccd) ?: return
```

[基本制御フロー: GATT準備](common/flows/prepare-gatt.md)では説明を省いていましたが、キャラクタリスティックにはUUID、属性、値の他、**ディスクリプタ**という、そのキャラクタリスティックに関する追加情報リストが備わっています。ディスクリプタはキャラクタリスティックと同様に、UUIDと値を持っています。

CCCDは、BLEの規格で決められているものであり、そのキャラクタリスティックからのNotification発生を許可するかどうかを制御するフラグを持っています。

CCCDのUUIDは全てのBLE製品で共通ですが、Android BLEフレームワークではこのUUIDオブジェクトが公開されていないため、BLEアプリ側で都度用意する必要があります。

```kotlin
object BluetoothLowEnergySpec {
    object GattUuid {
        val cccd
            = UUID.fromString("00002902-0000-1000-8000-00805f9b34fb")!!
    }
}
```

4. `dataFromPeripheralCccd`の`value`に`BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE`をセットしてから、[`BluetoothGatt.writeDescriptor()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#writeDescriptor(android.bluetooth.BluetoothGattDescriptor) )を実行します。

```kotlin
dataFromPeripheralCccd.value = BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE

val succeeded = gatt.writeDescriptor(dataFromPeripheralCccd)

if (!succeeded) {
    // TODO: 失敗時の処理
    return
}
```

この操作によってようやくLINBLE側にNotification許可イベントが走り、その通信結果が[`BluetoothGattCallback.onDescriptorWrite()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html#onDescriptorWrite(android.bluetooth.BluetoothGatt,%20android.bluetooth.BluetoothGattDescriptor,%20int) )で通知されます。

これでようやくLINBLEとの通信準備が完了します。


#### LINBLEへのデータ送信 - Write

> 参考: BLEとLINBLEの基本制御フロー: LINBLEとの双方向通信: [LINBLEへのデータ送信 - Write](common/flows/communicate-with-linble#linbleへのデータ送信-write)

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

##### セットしたデータ長がMTUを超えていた場合、超過分のデータは破棄される

Androidでのデータ送信では、[基本制御フロー](common/flows/communicate-with-linble#linbleへのデータ送信-write)で説明した通り、MTUによるデータ分割の実装は必須です。

Android OSにデータ送信を要求すると、その要求結果が[`BluetoothGattCallback.onCharacteristicWrite()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGattCallback.html#onCharacteristicWrite(android.bluetooth.BluetoothGatt,%20android.bluetooth.BluetoothGattCharacteristic,%20int) )で通知されますので、これをトリガーとして次の分割データを送るように繰り返し処理を構築します。



#### LINBLEからのデータ受信 - Notification

> 参考: BLEとLINBLEの基本制御フロー: LINBLEとの双方向通信: [LINBLEからのデータ送信 - Notification](common/flows/communicate-with-linble#linbleからのデータ送信-notification)

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



#### BLE制御の終了

> 参考: BLEとLINBLEの基本制御フロー: [BLE制御の終了](common/flows/stop-all-controls.md)

BLEスキャンを停止するためには、[`BluetoothLeScanner.stopScan()`]( https://developer.android.com/reference/android/bluetooth/le/BluetoothLeScanner.html#stopScan(android.bluetooth.le.ScanCallback) )を呼び出します。

このとき、スキャン開始のために使用した`ScanCallback`を渡す必要があります。

BLE通信の切断をするためには、[`BluetoothGatt.close()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#close() )を呼び出します。

`close()`を呼び出した`BluetoothGatt`オブジェクトは機能しなくなり、その後、いかなる`BluetoothGattCallback`イベントも発生しなくなります。

##### 切断APIが2種類あるが、close()を使うべき

BLE通信を切断するAPIは他にも[`BluetoothGatt.disconnect()`]( https://developer.android.com/reference/android/bluetooth/BluetoothGatt.html#disconnect() )がありますが、**弊社ではこちらは利用非推奨と考えています**。

!> **非推奨としている理由:** `disconenct()`で切断した場合、`BluetoothGattCallback.onConnectionStateChange()`が発生します。多くの場合、切断という行為において、その操作の完了をアプリが待機しなければならない道理はありません。
切断はOS内部のバックグラウンド動作に任せてしまい、アプリでは次の処理に移った方が簡単と言えます。
また、`onConnectionStateChange()`は通信異常による切断時にも発生するため、その切断イベントが自分から切断した正常系によるものなのか、相手から切られた異常系によるものなのかを判断する必要が生じ、コードを余計に複雑にしてしまいます。




#### その他特記事項

##### GATT操作APIではタイムアウト管理を行うべき

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


##### BLE接続後、ユーザーによってBluetooth機能がオフにされた場合、OSからのエラー通知が一切発生しない

特殊なケースですが、通信中にユーザーによってBluetooth機能がオフにされた場合、LINBLE側は正常に切断されますが、`BluetoothGattCallback.onConnectionStateChange()`は発生しないという挙動になります。

このため、[端末の状態の確認](#端末の状態の確認)で紹介したBluetooth状態の監視機構の搭載が重要となります。

サンプルコードではBluetooth機能がオフにされたら、通信中のBLEオブジェクトを破棄するように実装しています。


##### スキャンを30分間行い続けていると、日和見スキャンモードに格下げされる

[`BluetoothLeScanner`]( https://developer.android.com/reference/android/bluetooth/le/BluetoothLeScanner )が日和見スキャンモード（[`ScanSettings.SCAN_MODE_OPPORTUNISTIC`]( https://developer.android.com/reference/android/bluetooth/le/ScanSettings.html#SCAN_MODE_OPPORTUNISTIC) ）に格下げされると、そのスキャナーへのアドバタイズ発見通知は基本的に発生しなくなり、他アプリがスキャンをした際についでに通知してくれる、という奇妙な動作状態になります。

長時間スキャンを行う必要があるアプリは、スキャン開始後、例えば5分に1回など、定期的にスキャン処理を再起動するようにしてください。



##### スキャン開始を30秒以内に5回呼び出すとエラーになる

Android7より、スキャン開始・スキャン停止を30秒以内に5回呼び出すと、そのスキャン開始が内部で失敗するようになっています。
LogCatには以下のエラーログが記録されます（`Show only selected application`フィルタが有効になっていると表示されません。`No Filters`を選択してください）。

```
App 'xxx' is scanning too frequently
```

**厄介なことに、この時、[`ScanCallback.onScanFailed()`]( https://developer.android.com/reference/android/bluetooth/le/ScanCallback.html#onScanFailed(int) )によるスキャンエラー報告は発生してくれません**。したがって、ユーザーに対してスキャン開始ができなかったことを知らせることもできません。

このエラーは、意図的なスキャン開始・スキャン停止を繰り返すコードを実装してなくても、不意に遭遇してしまうことがあります。
例えば、画面遷移とスキャンの開始・停止を連動させている場合、ユーザーが画面を行ったり来たりするだけでこのエラー発生条件が満たされてしまいます。

?> サンプルコードでは、この問題に対する対策コードは搭載していません。対応策としては、スキャン開始前に「5回前のスキャン開始時刻」と「現在の時刻」の時間差を確認し、これが30秒以内だったらその差だけスキャン開始を遅延させるロジックを追加することが考えられます。

