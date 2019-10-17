# 資料内で使用するPlantUML図

資料に使用する全てのPlantUMLソースコードを、コードブロックで記述します。

!> **本ファイルは資料開発者向けに用意しているものです。PlantUMLのオンラインレンダリングサービスへの負荷となってしまうため、このページへのGitHub Pagesからのアクセスが行われないようにしてください。**

### 開発者向け情報

* 普段は[Markdown Preview Enhanced](https://shd101wyy.github.io/markdown-preview-enhanced/)からレンダリング後のPlantUML図を確認しながら開発します。
* 図の調整が終わり次第、VSCodeの[PlantUMLプラグイン](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml)で「ファイル内のダイアグラムをエクスポート」を使って、pngファイルへの一括ビルドを行ってください。
* [PlantUML公式のレンダリングWeb API](http://plantuml.com/ja/)には転送量制限があるため、開発中の積極使用は避け、ローカルの `plantuml.jar` が使用されるように設定してください。

## PlantUML

### 語句定義

```plantuml
@startuml deployment_device_components
left to right direction

mainframe 語句定義

node "制御対象" as Target
node "コントローラ" as Controller

Controller <==> Target: "UART通信"
@enduml
```



### 概略クラス図

```plantuml
@startuml classes_introduction_summary

mainframe 概略クラス図

left to right direction

hide empty field
hide empty method

interface Application #ffe0e0

class WirelessUartController #fefece {
}

interface BluetoothCentralController #e0f4ff {
}

interface "BLE API" as BLEAPI <<Platform SDK>>  #dddddd {
}

Application --> WirelessUartController: 機器の制御\nを要求
Application .r.> UartCommand: <<create>>
WirelessUartController --> BluetoothCentralController: BLE制御\nを要求
BluetoothCentralController --> BLEAPI: 各プラットフォーム\nごとの\nBLE制御

WirelessUartController .r.> UartCommand: <<use>>

@enduml
```



### BLE制御開始シーケンス

```plantuml
@startuml sequence_start

mainframe BLE制御開始シーケンス

participant ": Application" as Application #ffe0e0
participant ": WirelessUartController" as WirelessUartController
participant "current\n: Advertisement" as Advertisement #e0f4ff
participant ": BluetoothCentralController" as BluetoothCentralController #e0f4ff

autonumber

activate Application
Application -> WirelessUartController: start()

activate WirelessUartController

WirelessUartController -> BluetoothCentralController: startDeviceBluetoothStateMonitoring()
activate BluetoothCentralController

BluetoothCentralController ->]: 端末のBluetooth状態を監視
deactivate BluetoothCentralController
deactivate WirelessUartController
deactivate Application

BluetoothCentralController <-]: 端末のBluetooth状態の変更通知
activate BluetoothCentralController

BluetoothCentralController -> WirelessUartController: onChange(観測したBluetooth状態)
activate WirelessUartController

alt Bluetooth機能がON

ref over WirelessUartController, BluetoothCentralController: スキャン開始シーケンス

end

@enduml
```


### スキャン開始シーケンス

```plantuml
@startuml sequence_scan_advertisements

mainframe スキャン開始シーケンス

participant ": WirelessUartController" as WirelessUartController
participant "current\n: Advertisement" as Advertisement #e0f4ff
participant ": BluetoothCentralController" as BluetoothCentralController #e0f4ff

autonumber

activate WirelessUartController
WirelessUartController -> BluetoothCentralController: scanAdvertisements()
activate BluetoothCentralController
BluetoothCentralController ->]: スキャン開始
deactivate BluetoothCentralController
deactivate WirelessUartController

alt アドバタイズ受信時
    BluetoothCentralController <-]: アドバタイズ受信
    activate BluetoothCentralController #e7d9ff

    create Advertisement
    BluetoothCentralController -> Advertisement: <<create>>
    activate Advertisement #e7d9ff

    BluetoothCentralController -> WirelessUartController: onDiscover(current)
    activate WirelessUartController #e7d9ff
    note over WirelessUartController
    currentのフィールドを確認し、
    接続対象のLINBLEからのものであるかを判別する
    end note

    alt 接続対象のアドバタイズを獲得
        WirelessUartController -> BluetoothCentralController: cancelScan()
        activate BluetoothCentralController #e7d9ff
        BluetoothCentralController ->]: スキャン停止
        deactivate BluetoothCentralController

        ref over WirelessUartController, BluetoothCentralController: 接続対象とのBLE接続シーケンス
    end

    deactivate WirelessUartController
    deactivate BluetoothCentralController
end

@enduml
```


## 接続対象とのBLE接続シーケンス

```plantuml
@startuml sequence_connection

mainframe 接続対象とのBLE接続シーケンス

participant ": WirelessUartController" as WirelessUartController
participant "connected\n: Linble" as Linble #e0f4ff
participant ": BluetoothCentralController" as BluetoothCentralController #e0f4ff

autonumber

activate WirelessUartController #dac4ff

WirelessUartController -> BluetoothCentralController: connect(target)
activate BluetoothCentralController #dac4ff

BluetoothCentralController ->]: 対象とのBLE接続を開始

deactivate BluetoothCentralController

...

BluetoothCentralController <-]: 接続結果通知
activate BluetoothCentralController #e7d9ff

break 失敗時

BluetoothCentralController -> WirelessUartController: onError(失敗理由)
activate WirelessUartController #e7d9ff

note over WirelessUartController #ffb3b3: スキャンからやり直す

deactivate WirelessUartController

end

create Linble
BluetoothCentralController -> Linble: <<create>>
activate Linble

BluetoothCentralController -> WirelessUartController: onSuccess(connected)
activate WirelessUartController #e7d9ff

ref over WirelessUartController,Linble,BluetoothCentralController: GATT準備シーケンス

deactivate WirelessUartController


deactivate WirelessUartController

deactivate BluetoothCentralController

@enduml
```





### GATT準備シーケンス

#### GATTとは

```plantuml
@startuml classes_what_is_gatt

mainframe GATTとは

hide empty method
hide empty field

interface "GATTクライアント" as GattClient #fbbdff {
}

interface "GATTサーバー" as GattServer #fbbdff {
}

class "プロファイル" as GattProfile #c0bdff {
}

class "サービス" as GattService #c0bdff {
    - UUID
}

class "キャラクタリスティック" as GattCharacteristic #c0bdff {
    - UUID
    - 属性
    - 値
}

GattClient -r-> GattServer: 接続

GattClient ..> GattProfile: 利用
GattServer --> "*" GattProfile: 定義\nおよび\n提供

GattProfile --> "1..*" GattService
GattService -r-> "*" GattCharacteristic

GattProfile -[hidden]- GattCharacteristic

@enduml
```



#### サービス・キャラクタリスティックの検索

```plantuml
@startuml sequence_discover_services_and_characteristics

mainframe サービス・キャラクタリスティックの検索

participant ": WirelessUartController" as WirelessUartController
participant "connected\n: Linble" as Linble #e0f4ff
participant ": BluetoothCentralController" as BluetoothCentralController #e0f4ff

autonumber

==サービス検索==

activate WirelessUartController #dac4ff
WirelessUartController -> Linble: discoverServices()
activate Linble
Linble ->]: サービス検索

...

Linble <-]: サービス検索結果通知

break 失敗時

WirelessUartController <- Linble: onError(失敗理由)
activate WirelessUartController #e7d9ff
note over WirelessUartController #ffb3b3: 切断し、\nスキャンからやり直す
deactivate WirelessUartController


end

WirelessUartController <- Linble: onSuccess(this)
activate WirelessUartController #e7d9ff

==キャラクタリスティック検索==

WirelessUartController -> Linble: discoverCharacteristics()
activate Linble
Linble ->]: キャラクタリスティック検索
deactivate WirelessUartController

deactivate Linble

...

Linble <-]: キャラクタリスティック検索結果通知

break 失敗時

WirelessUartController <- Linble: onError(失敗理由)
activate WirelessUartController #e7d9ff
note over WirelessUartController #ffb3b3: 切断し、\nスキャンからやり直す
deactivate WirelessUartController

end

note over Linble: LINBLE GATT Serviceの\nキャラクタリスティックオブジェクトを\n保持

WirelessUartController <- Linble: onSuccess(this)
activate WirelessUartController #e7d9ff

ref over WirelessUartController, Linble: DataFromPeripheralに対するNotificationの許可

@enduml
```


#### DataFromPeripheralに対するNotificationの許可

```plantuml
@startuml sequence_enable_notification_from_dfp

mainframe DataFromPeripheralに対するNotificationの許可

participant ": WirelessUartController" as WirelessUartController
participant "connected\n: Linble" as Linble #e0f4ff
participant ": BluetoothCentralController" as BluetoothCentralController #e0f4ff

autonumber

==Notification許可==
activate WirelessUartController #dac4ff

WirelessUartController -> Linble: enableNotification()
activate Linble
Linble ->]: Notification許可

deactivate Linble

...

Linble <-]: Notification許可結果通知

break 失敗時

WirelessUartController <- Linble: onError(失敗理由)
activate WirelessUartController #e7d9ff
note over WirelessUartController #ffb3b3: 切断し、\nスキャンからやり直す
deactivate WirelessUartController

end

WirelessUartController <- Linble: onSuccess(this)
activate WirelessUartController #e7d9ff

note over WirelessUartController, Linble: LINBLEとの双方向通信を開始

@enduml
```




### データ送信シーケンス

```plantuml
@startuml sequence_send_write_request

mainframe データ送信シーケンス

participant ": Application" as Application #ffe0e0
participant "command\n: UartCommand" as UartCommand #ccffd5
participant ": WirelessUartController" as WirelessUartController
participant ": TxPacketDivider" as TxPacketDivider
participant "connected\n: Linble" as Linble #e0f4ff

autonumber

activate Application

create UartCommand
Application -> UartCommand: <<create>>
activate UartCommand

Application -> WirelessUartController: write(command)

activate WirelessUartController #dac4ff

create TxPacketDivider
WirelessUartController -> TxPacketDivider: <<create>>(command.toByteArray(), 分割サイズ=20)

destroy UartCommand
deactivate Application

loop MTU分割バイト列の数だけ

WirelessUartController -> TxPacketDivider: next()
activate TxPacketDivider

autonumber stop
return MTU分割バイト列
autonumber resume

WirelessUartController -> Linble: write(MTU分割バイト列)
activate Linble
Linble ->]: データ送信(dataToPeripheral, MTU分割バイト列)
deactivate Linble

deactivate WirelessUartController

Linble <-]: 送信結果通知
activate Linble

break 失敗時

WirelessUartController <- Linble: onError(失敗理由)
activate WirelessUartController #e7d9ff
autonumber stop
Application <- WirelessUartController: 書き込み失敗通知
activate Application
deactivate Application
autonumber resume
deactivate WirelessUartController

end

WirelessUartController <- Linble: onSuccess(this)
activate WirelessUartController #e7d9ff

break 最後の書き込み
autonumber stop
Application <- WirelessUartController: 書き込み成功通知
activate Application
deactivate Application
autonumber resume
end

deactivate WirelessUartController

end

deactivate Linble
destroy TxPacketDivider

@enduml
```




### Notification受信シーケンス

```plantuml
@startuml sequence_handle_notification

mainframe Notification受信シーケンス

participant ": WirelessUartController" as WirelessUartController
participant "rxPacket\n: UartRxPacket" as UartRxPacket #f6ff9c
participant ": UartDataParser" as UartDataParser
participant "current\n: NotificationEvent" as NotificationEvent #e0f4ff
participant "connected\n: Linble" as Linble #e0f4ff

autonumber

...

Linble <-]: データ受信通知
activate Linble

create NotificationEvent
Linble -> NotificationEvent: <<create>>
activate NotificationEvent #e7d9ff

Linble -> WirelessUartController: onNotify(current)
activate WirelessUartController #e7d9ff

WirelessUartController -> UartDataParser: parse(current.data)
activate UartDataParser
ref over UartDataParser: Notification受信データ解析フロー
alt 解析成功
create UartRxPacket
UartDataParser -> UartRxPacket: <<create>>
activate UartRxPacket
UartDataParser -> WirelessUartController: onParse(rxPacket)
destroy UartRxPacket
end
deactivate UartDataParser
deactivate WirelessUartController

destroy NotificationEvent

@enduml
```


#### Notification受信データ解析フロー

```plantuml
@startuml activity_parse_rx_data

mainframe Notification受信データ解析フロー

start
:rxDataBufferの末尾に\n新規受信データを追加する;

repeat
    if (rxDataBufferのindex=0から1個のデータを参照し、\nlengthとする) then (バッファ不足)
        stop
    else (else)
    endif

    if (rxDataBufferのindex=1からlength個のデータを参照し、\nfollowingとする) then (バッファ不足)
        stop
    else (else)
    endif

    :rxDataBufferからlengthとfollowing部分のデータを削除する;

    :followingのindex=0から1個のデータを取り出し、\nrxTypeとする;

    :followingのindex=1からlength-1個のデータを取り出し、\nrxPayloadとする;

    if (rxTypeと一致するtypeを持つ\nUartRxPacketのサブクラスを特定し、\nそのオブジェクトを生成する) then (rxTypeがどのtypeとも一致しない)
        
    else (else)
        :生成オブジェクトを解析結果として上層へ通知;
    endif
repeat while (rxDataBufferの長さを確認) is (空でない)

stop
@enduml
```




## BLE制御の停止

```plantuml
@startuml sequence_stop

mainframe BLE制御の停止シーケンス

participant ": Application" as Application #ffe0e0
participant ": WirelessUartController" as WirelessUartController
participant ": BluetoothCentralController" as BluetoothCentralController #e0f4ff
participant "connected\n: Linble" as Linble #e0f4ff

autonumber

...

Application -> WirelessUartController: stop()
activate WirelessUartController

WirelessUartController -> BluetoothCentralController: cancelScan()
activate BluetoothCentralController
BluetoothCentralController ->]: スキャン停止
deactivate BluetoothCentralController

WirelessUartController -> BluetoothCentralController: cancelConnection()
activate BluetoothCentralController
BluetoothCentralController ->]: BLE接続試行をキャンセル
deactivate BluetoothCentralController

WirelessUartController -> Linble: cancelConnection()
activate Linble
Linble ->]: 対象とのBLE接続を終了
destroy Linble

@enduml
```





### BLE制御構造


```plantuml
@startuml classes_ble-control

mainframe BLE制御用構造

left to right direction

class WirelessUartController #fefece {
    - targetLinbleAddress: String
    + start()
    + write(UartCommand)
    + stop()
}

class TxPacketDivider #fefece {
    - fragmentedByteArrayList: List<ByteArray>
    + constructor(data: ByteArray, size: Int)
    + next(): ByteArray?
}

class UartDataParser #fefece {
    + rxDataBuffer: ByteArray
    + constructor(UartDataParserCallback)
    + parse(ByteArray)
}

interface UartDataParserCallback #fefece {
    + onParse(UartRxPacket)
}

enum DeviceBluetoothState #e0f4ff {
    + Unknown
    + PoweredOff
    + PoweredOn
}

hide DeviceBluetoothState method

interface DeviceBluetoothStateMonitoringCallback #e0f4ff {
    fun onChange(deviceBluetoothState: DeviceBluetoothState)
}

interface BluetoothCentralController #e0f4ff {
    ..端末のBluetooth状態の監視..
    + startDeviceBluetoothStateMonitoring(callback: DeviceBluetoothStateMonitoringCallback)
    + stopDeviceBluetoothStateMonitoring()
    ..スキャン..
    + scanAdvertisements(ScanAdvertisementsCallback)
    + cancelScan()
    ..接続..
    + connect(target: Advertisement, GattOperationCallback)
    + isConnected: Boolean {readonly}
    + cancelConnection()
}

interface ScanAdvertisementsCallback #e0f4ff {
    + didDiscover(Advertisement)
}

interface GattOperationCallback #e0f4ff {
    + onError(reason: Exception)
    + onSuccess(linble: Linble)
}

interface Advertisement #e0f4ff {
    + deviceName: String
    + deviceAddress: String
}

interface Linble #e0f4ff {
    - dataFromPeripheral: GattCharacteristic?
    - dataToPeripheral: GattCharacteristic?
    ..GATT操作..
    + discoverServices(GattOperationCallback)
    + discoverCharacteristics(GattOperationCallback)
    + enableNotification(GattOperationCallback, NotificationCallback)
    + write(ByteArray, GattOperationCallback)
    ..切断..
    ~ cancelConnection()
}

interface NotificationCallback #e0f4ff {
    + onNotify(NotificationEvent)
}

interface NotificationEvent #e0f4ff {
    + data: ByteArray
}

WirelessUartController -r-> "0..1" TxPacketDivider

WirelessUartController --> "1" DeviceBluetoothStateMonitoringCallback
DeviceBluetoothStateMonitoringCallback ..> DeviceBluetoothState

WirelessUartController -r-> "1" UartDataParser
UartDataParser --> "0..1" UartDataParserCallback
WirelessUartController --> "1" UartDataParserCallback
WirelessUartController --> "1" BluetoothCentralController
WirelessUartController --> "1" ScanAdvertisementsCallback
WirelessUartController --> "1" GattOperationCallback: connectCallback
WirelessUartController --> "1" GattOperationCallback: discoverCallback
WirelessUartController --> "1" GattOperationCallback: enableNotificationCallback
WirelessUartController --> "1" NotificationCallback
WirelessUartController --> "0..1" Linble: connected

BluetoothCentralController -l-> "1" DeviceBluetoothState: currentDeviceBluetoothState

BluetoothCentralController -u-> "0..1" DeviceBluetoothStateMonitoringCallback
BluetoothCentralController -u-> "0..1" ScanAdvertisementsCallback
BluetoothCentralController -u-> "0..1" GattOperationCallback
BluetoothCentralController ..> Advertisement: <<create>>\nスキャン結果として生成
BluetoothCentralController --> "0..1" Linble: connecting

Linble ..> NotificationEvent: <<create>>\nデータ受信結果として生成
Linble -u-> "0..1" GattOperationCallback
Linble -u-> "0..1" NotificationCallback

BluetoothCentralController -r[hidden]- Linble



@enduml
```


### UARTコマンドインタフェース

#### コマンド側

```plantuml
@startuml classes_uart-command-interface-command

mainframe UARTコマンド

left to right direction

abstract class UartCommand #ccffd5 {
    # {abstract} length: Byte
    # {abstract} type: Byte
    # {abstract} createPayload(): ByteArray?
    + toByteArray(): ByteArray
}
note as noteTx #ccffd5
    端末からの
    データ送信時に利用
end note
class UartCommandConnectionTest #ccffd5 {
    + length: Byte = 1
    + type: Byte = 0x00
}
note right #ccffd5
導通テスト.コマンド
end note
class UartCommandRegisterRead #ccffd5 {
    + length: Byte = 2
    + type: Byte = 0x01
    + registerNumber: RegisterNumber
    + constructor(registerNumber: RegisterNumber)
}
note right #ccffd5
レジスタ値アクセス.取得.コマンド
end note
class UartCommandRegisterWrite #ccffd5 {
    + length: Byte = 3
    + type: Byte = 0x02
    + registerNumber: RegisterNumber
    + hexdecimal: Byte
    + constructor(registerNumber: RegisterNumber, hexdecimal: Byte)
}
note right #ccffd5
レジスタ値アクセス.設定.コマンド
end note
class UartCommandVersionRead #ccffd5 {
    + length: Byte = 1
    + type: Byte = 0x03
}
note right #ccffd5
ホストマイコンバージョン確認.コマンド
end note
class UartCommandSensorSampling #ccffd5 {
    + length: Byte = 2
    + type: Byte = 0x04
    + duration: DurationIntervalSeconds
    + constructor(duration: DurationIntervalSeconds)
}
note right #ccffd5
センササンプリング実行要求.コマンド
end note
class UartCommandDeviceNameRead #ccffd5 {
    + length: Byte = 1
    + type: Byte = 0x05
}
note right #ccffd5
デバイス名アクセス.取得.コマンド
end note
class UartCommandDeviceNameWrite #ccffd5 {
    + length: Byte = 1 + deviceName.size
    + type: Byte = 0x06
    + deviceName: AsciiString
    + constructor(deviceName: AsciiString)
}
note right #ccffd5
デバイス名アクセス.設定.コマンド
end note

UartCommandConnectionTest -u-|> UartCommand
UartCommandRegisterRead -u-|> UartCommand
UartCommandRegisterWrite -u-|> UartCommand
UartCommandVersionRead -u-|> UartCommand
UartCommandSensorSampling -u-|> UartCommand
UartCommandDeviceNameRead -u-|> UartCommand
UartCommandDeviceNameWrite -u-|> UartCommand

noteTx .. UartCommand

@enduml
```



#### レスポンス・イベント側

```plantuml
@startuml classes_uart-command-interface-response-event

mainframe レスポンス・イベント側

left to right direction

abstract class UartRxPacket #f6ff9c {
    + constructor(rxPayload: ByteArray)
}

abstract class UartResponse #ffebcc
note as noteRx #ffebcc
    LINBLEからの
    データ受信時に利用
end note
class UartResponseConnectionTest #ffebcc {
    + {static} length: Byte = 1 {readonly}
    + {static} type: Byte = 0x40 {readonly}
}
note right #ffebcc
導通テスト.レスポンス
end note
class UartResponseRegisterRead #ffebcc {
    + {static} length: Byte = 3 {readonly}
    + {static} type: Byte = 0x41 {readonly}
    + registerNumber: RegisterNumber
    + hexdecimal: Byte
    + constructor(registerNumber: RegisterNumber, hexdecimal: Byte)
}
note right #ffebcc
レジスタ値アクセス.取得.レスポンス
end note
class UartResponseRegisterWrite #ffebcc {
    + {static} length: Byte = 1 {readonly}
    + {static} type: Byte = 0x42 {readonly}
}
note right #ffebcc
レジスタ値アクセス.設定.レスポンス
end note
class UartResponseVersionRead #ffebcc {
    + {static} type: Byte = 0x43 {readonly}
    + version: AsciiString
    + constructor(version: AsciiString)
}
note right #ffebcc
ホストマイコンバージョン確認.レスポンス
（可変長）
end note
class UartResponseSensorSampling #ffebcc {
    + {static} length: Byte = 1 {readonly}
    + {static} type: Byte = 0x44 {readonly}
}
note right #ffebcc
センササンプリング実行要求.レスポンス
end note

class UartResponseDeviceNameRead #ffebcc {
    + type: Byte = 0x45 {readonly}
    + deviceName: AsciiString
    + constructor(deviceName: AsciiString)
}
note right #ffebcc
デバイス名アクセス.取得.レスポンス
end note
class UartResponseDeviceNameWrite #ffebcc {
    + length: Byte = 1 {readonly}
    + type: Byte = 0x46
}
note right #ffebcc
デバイス名アクセス.設定.レスポンス
end note


abstract class UartEvent #ffe3fa
class UartEventSensorSampling #ffe3fa {
    + {static} type: Byte = 0x84 {readonly}
    + state: SamplingState
    + value: Float?
    + constructor(state: SamplingState, value: Float?)
}
note right #ffe3fa
センササンプリング実行要求.イベント
（可変長）
end note


UartResponse -u-|> UartRxPacket
UartResponseConnectionTest -u-|> UartResponse
UartResponseRegisterRead -u-|> UartResponse
UartResponseRegisterWrite -u-|> UartResponse
UartResponseVersionRead -u-|> UartResponse
UartResponseSensorSampling -u-|> UartResponse
UartResponseDeviceNameRead -u-|> UartResponse
UartResponseDeviceNameWrite -u-|> UartResponse

UartEvent -u-|> UartRxPacket
UartEventSensorSampling -u-|> UartEvent

noteRx .. UartResponse
noteRx .. UartEvent

@enduml
```



#### データクラス

```plantuml
@startuml classes_datatype

mainframe データクラス

class RegisterNumber {
        + 値: Byte {0 .. 7}
    }

class AsciiString {
    + 文字列: String {ASCII印字可能文字のみ}
}

class DurationIntervalSeconds {
    + 値: Byte {1 .. 60}
}

enum SamplingState <<Byte>> {
    + Stopped = 0
    + Sampling = 1
}

@enduml
```


