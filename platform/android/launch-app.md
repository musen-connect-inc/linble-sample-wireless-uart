# Android版サンプルコード - アプリの起動

{docsify-updated}

アプリを起動させると、`MainActivity`が立ち上がり、Android OSから`onCreate()`や`onStart()`がコールされます。

`onCreate()`時、`MainActivityViewModel`内の`RuntimePermissionHandler`によって、その時点の[Runtime Permission]( #runtime-permission取得 )の付与状況の確認が行われます。

?> `MainActivityViewModel`は、画面回転時の`Activity`再生成の影響を受けません。ここにBLE制御を管理するオブジェクトを所持させることで、BLE処理が継続するようになっています。逆に言えば、`Activity`にBLE制御担当オブジェクトを所持させてしまうと、画面回転のたびにBLE通信が切断されるような動きになってしまいます。


## Runtime Permission取得

[Runtime Permission]( https://developer.android.com/training/permissions/requesting?hl=ja )はAndroid 6で導入された、「アプリ起動後におけるデバイス機能の利用認可の仕組み」です。
Android SDKの一部APIを利用するためには、Google Playからのインストール時点とは別に、アプリ動作中にユーザーと対話をしてそのAPIの利用許可を得る必要があります。

Androidバージョンに応じて必要な権限が変わります。

### Android12以降

Android12以降ではBluetooth権限として、[`Manifest.permission.BLUETOOTH_SCAN`]( https://developer.android.com/reference/android/Manifest.permission#BLUETOOTH_SCAN )、[`Manifest.permission.BLUETOOTH_ADVERTISE`]( https://developer.android.com/reference/android/Manifest.permission#BLUETOOTH_ADVERTISE )、[`Manifest.permission.BLUETOOTH_CONNECT`]( https://developer.android.com/reference/android/Manifest.permission#BLUETOOTH_CONNECT )が導入されています。
アプリが使用する機能の権限のみRuntime Permission取得を行います。

加えて、**アプリがスキャン結果を使用して位置情報を取得するかどうかに応じて、追加で作業が必要になります。**

位置情報が必要な場合は、[`Manifest.permission.ACCESS_FINE_LOCATION`]( https://developer.android.com/reference/android/Manifest.permission#ACCESS_FINE_LOCATION )権限が追加で必要になります。

位置情報が必要ない場合は、
`android:usesPermissionFlags`属性を`BLUETOOTH_SCAN`権限宣言に追加し、この属性の値を`neverForLocation`に設定します。

```xml
<uses-permission
    android:name="android.permission.BLUETOOTH_SCAN"
    android:usesPermissionFlags="neverForLocation" />
```


### Android12未満

Android12未満では、**Androidの思想において、Bluetooth機能は「位置情報」機能のカテゴリの中に入れられており、**Bluetooth系のAPIを利用するためには[`Manifest.permission.ACCESS_FINE_LOCATION`]( https://developer.android.com/reference/android/Manifest.permission#ACCESS_FINE_LOCATION )のRuntime Permission取得が必要です。

?> ユーザーは「位置情報」という機能について、**「端末のGPS情報を記録・外部送信して悪用されるかもしれない」**といったネガティブな印象を持っている可能性があり、位置情報機能を要求してきたアプリに対して不信感を抱くかもしれません。Runtime Permissionを取得する前に、「Bluetooth機能の利用のために位置情報利用権限が必要である」「得られた権限はBLE通信のために利用し、位置の追跡のためには使用しない」ということを説明すべきです。サンプルコードでは、Runtime Permissionの取得用システムダイアログを表示する前に、この点についてユーザーへ`Text`コンポーザブルを使用して説明しています。


## BLE制御の開始

Runtime Permissionが取得済みである場合、`WirelessUartController.start()`がコールされ、BLE処理が開始されます。

以下、[BLEとLINBLEの基本制御フロー](common/flows/introduction.md)で登場していた`BluetoothCentralController`クラスで対応した処理が行われていきます。
