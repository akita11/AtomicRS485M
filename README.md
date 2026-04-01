# Atomic RS485M Base

<img src="https://github.com/akita11/AtomicRS485M/blob/main/AtomicRS485M-1.jpg" width="240px">

<img src="https://github.com/akita11/AtomicRS485M/blob/main/AtomicRS485M-2.jpg" width="240px">

<img src="https://github.com/akita11/AtomicRS485M/blob/main/AtomicRS485M-F.jpg" width="240px">

RS485規格の半二重シリアル通信を行うことができるAtomic Baseです。
M5StackのATOMシリーズを載せて使用します。

[M5Stack社のATOMIC RS485ベース](https://www.switch-science.com/products/9180)とは異なり、本来のRS485の通信仕様でマルチドロップの1:N通信トポロジを構成できます（詳細は後述）。
データを送信している期間のみ伝送路を送信駆動する制御を行うことができます（通信速度は変更可能）。

また伝送路側から最大24Vを供給してマイコン側に5Vを供給することができます。

理論上最大256台まで接続可能な低入力容量型のRS485ドライバを使用しています。


## RS485通信の基本構成と「M5Stack社のATOMIC RS485ベース」との相違点

<img src="https://github.com/akita11/RS485M_Unit/blob/main/config.png" width="320px">

RS485は、2本の伝送路(Line)を使ってシリアルデータ(UART)を送受信する規格です。
規格の詳細と、M5Stack社のATOMIC RS485ベースでの留意すべき点は[こちら](https://note.com/akita11/n/n0c082069f98c)を参照してください。

「M5Stack社のATOMIC RS485ベース」との相違点は以下です。
（以下のカッコ内の※以下は「M5Stack社のATOMIC RS485ベース」の仕様）

- 非送信時には伝送路が非駆動（※"1"で駆動）
- 送信時のみ伝送路を駆動、1バイト単位自動またはGPIOで制御、自動制御にも設定可能（※常時"1"で駆動）
- 給電入力は48Vまで、24V給電時の過渡応答でも破損しにくい（※給電入力の絶対最大定格が30Vのため24V給電時の過渡応答で破損する場合がある）


## 使い方

基本的には[M5Stack社のATOMIC RS485ベース](https://www.switch-science.com/products/9180)と同じ使い方ですが、送信制御のみ異なり、以下の通りの仕様です。

なおJP5をショートすると伝送路の終端抵抗（120Ω）をONにできます。
またJP6とJP7をショートすると、伝送路のプルアップ・プルダウンを有効にできます（Aプルアップ、Bがプルダウン（5.1kΩ））。


### データ送信

初期設定時には、TXen（ATOM Lite/Matrixの場合はGPIO23）でDriverの有効／無効を制御できます。

1:1接続時の単方向（1つが送信専用、もう1つが受信専用）の場合の送信側は、常時有効（TXen=1）でOKです。

1:N接続時（複数の回路が伝送路Lineにつながる）は、送信デバイスのみがTXen=1としてデータを送信し、その他のデバイスはTXen=0ととすることで、半二重通信が実現されます。

伝送路Receiverは、TXen=1（送信時）のみ無効となりますが、JP8を2-3側に切り替えると、伝送路レシーバReceiverを常時有効に設定できますが、この場合は、自分のDriverが送信したデータを常時受信することになります。


### 伝送路ドライバの自動制御

JP4をショートすると、Driverの自動制御が有効となります。
※このときATOM側のTXen（ATOM Lite/Matrixの場合はGPIO23）は入力設定としてください。

スタートビット検出から1バイト分（10bit分=STARTビット+データ8ビット+STOPビット）の時間だけ自動的にDriverが有効となり、伝送路にデータを送信します（伝送路を駆動中はLEDが点灯）。
このとき、Receiverは無効となり、データを受信しません。
通信方式はN81（8ビット・STOPビット1ビット、パリティなし）で固定です。

データ送信時にDriverを有効とする時間は、データ転送速度（ボーレート）に依存するため、後述の方法で変更できます


### 伝送速度（ボーレート）の変更

ATOM側からBREAK信号（一定時間以上、データ線を"L"に保持）することで、伝送速度変更モードに入り、LEDが点滅します。
この状態で、設定したい伝送速度（ボーレート）で以下のいずれか1バイトを送信することで、その波形から伝送速度を検出し、伝送速度の設定（Driverを有効とする時間）を変更します。
設定された伝送速度は電源を切っても保存されます。

- 'U' (0x55) : 1バイト伝送モード。1バイト伝送終了後の伝送路ドライバがDisableに戻ります。（LEDが3回点滅）
- '5' (0x35) : 連続伝送モード。送信開始語、一定時間データが送られなかった場合に伝送路ドライバがDisableに戻ります。（LEDが5回点滅）

なおBREAK信号は、ATOM等で生成、またはATOM等経由でPC用ターミナルソフトでも送出ができます（TeraTermではAlt+B、MacのscreenコマンドではCtrl+A→Ctr+B）。
[Groveケーブル-USBシリアル接続ケーブル](https://www.switch-science.com/products/9460)などを使って、PCと接続して設定変更や送受信テストができます。


### 電源給電の設定

<img src="https://github.com/akita11/AtomicRS485M/blob/main/AtomicRS485M-B.jpg" width="240px">

デフォルトでは、外部からの12/24V給電から本体とATOMへ5Vを供給します。
JP1とJP9で給電方法を変更できます。

- JP9: 給電方法
  - 1-2間: 外部12/24Vから5Vを生成し、本体とATOMへ供給（デフォルト）
  - 2-3間: 外部給電をそのまま本体・ATOMへ接続（※この場合は外部からの給電電圧は5Vとしてください。5V以上の電圧を加えると破損する場合があります）

- JP1: RS485ドライバの電源電圧の変更
  - 1-2間=3.3V（デフォルト）
  - 2-3間=5V


## Author

Junichi Akita (@akita11) / akita@ifdl.jp
