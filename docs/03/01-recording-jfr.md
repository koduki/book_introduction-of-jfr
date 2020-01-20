# 3.1 JDK Flight Recorderの記録

## JFRの取得

JFRを取得するには以下のいずれかの方法でJFRの記録を開始する必要があります。

1. GUI(JMC/VisualVM)で記録する
2. CLIで記録する
3. JVMオプションで記録する
4. プログラム上から記録する

実運用として一番利用するのはJVMオプションでの指定だと思いますが他の方法に関しても説明します。

## GUI(JMC/VisualVM)で記録する

JDK Mission ControlまたはVisualVMを利用することでJFRの記録をGUIから開始することが出来ます。
GUIから起動でき、結果もそのままJMC等で分析できるので従来的なプロファイラと同様の使い方をすることが出来ます。

## CLIで記録する

jcmdコマンドを使ってコマンドラインからJFRの記録を開始することが出来ます。
jcmdコマンドは１章でも説明したとおり主に`JFRの操作`, `プロセスの取得`, `スレッドダンプの取得`等が行えるJavaの診断ツールです。

JFRに関連するオプションは以下となります。

| コマンド | 説明|
| -- | -- |
| jcmd {引数なし}| Javaのプロセス一覧を表示する|
| jcmd {プロセスID} JFR.start {JFRオプション}| JFRの記録を開始する |
| jcmd {プロセスID} JFR.dump {JFRオプション}| 実行中の記録を停止し循環バッファより記録内容をファイル出力する |
| jcmd {プロセスID} JFR.check | JFRの実行情報を表示する |
| jcmd {プロセスID} JFR.stop | JFRの記録を停止する |

オプションなしのjcmdまたはPSコマンドでターゲットのプロセスIDを確認して`JFR.start`などを実行します。
例えばプロセスIDが1234のJavaプロセスで１分感の記録を開始してmyrecording.jfrに保存するには以下のコマンドを実行します。

```bash
$ jcmd 1234 JFR.start duration=1m filename=myrecording.jfr
```

実際にはJFRの起動は後述するJVMオプションで指定して、JFRファイルを取得したい任意のタイミングでdumpを使ってファイル出力をさせるのが実運用では多いでしょう。

```bash
$ jcmd 1234 JFR.dump filename=myrecording.jfr
```

`JFR.dump`を使うことでJFRの記録自体はメモリ上に常時しつつ、必要なタイミングでファイルに出力するという運用が出来ます。

## JVMオプションで記録する

JFRの記録の開始として最も頻繁に利用するのがJVMオプションでの指定です。

JFRはやはりブラックボックス分析に向いた仕組みなので常時有効にしておくのが基本的な戦略となります。そうなるとGUIやコマンドで必要なときに付けるというよりも、JVMオプションで起動時にしていして常時稼働させておくのがベストプラクティスとなります。

JVMオプションでは`-XX:StartFlightRecording`と`-XX:FlightRecorderOptions`を使用します。`-XX:StartFlightRecording`は時間固定の場合の指定、`-XX:FlightRecorderOptions`は継続実行の時のオプションですが組み合わせで使う必要がある認識です。

注意点としてオプションがJDK8の頃とJDK9以降で変わっているということです。

例えば以下のJDK8とJDK11は同じ意味のオプションです。

JDK 8:

```bash
-XX:+UnlockCommercialFeatures
-XX:+FlightRecorder
-XX:FlightRecorderOptions=defaultrecording=true,
dumponexit=true, dumponexitpath=/var/log/myapp/myapp.jfr
```

JDK 11:

```bash
-XX:StartFlightRecording=dumponexit=true,
filename=/var/log/myapp/myapp.jfr
```

JDK11からはOpenJDKに寄贈されたのでJFRの利用に`UnlockCommercialFeatures`が不要になったというのもありますが、`-XX:+FlightRecorder`オプションが廃止されたり`dumponexitpath`が`filename`になったりと変わった部分も多いです。

そのためWebの資料や既存の運用スクリプトを参考にする時には注意してください。
最新のオプションは[Java SE 11 - Advanced Runtime Options](https://docs.oracle.com/en/java/javase/11/tools/java.html#GUID-3B1CE181-CD30-4178-9602-230B800D4FAE)で確認することができます。

JFRオプションはそれぞれ以下になります。

### XX:StartFlightRecording

`-XX:StartFlightRecording=parameter=value`はjcmdの`JFR.start`と等価なJFRを開始するためのオプションです。
カンマ区切りで多くのオプションを指定することが出来ます。

| オプション| デフォルト値| 説明|
| -- | -- | -- |
| delay=time | 0s | JFR記録の開始時間を遅らせます。起動時の処理を例外として記録したくない時に利用できます。秒なら`s`, 分なら`m`, 時間なら`h`を付けます。|
| disk={true, false} |false | 循環バッファの値をディスクに書き込むかどうかを指定します。|
| dumponexit={true, false} |false | JVMプロセスがシャットダウンした時にファイルにダンプします。ファイル名が指定されていない場合はプロセスの起動したディレクトリに自動的に生成された名前で出力されます。自動的に生成される名前は「プロセスID」「レコーディングID」「タイムスタンプ」を含んでいます。 例: hotspot-pid-47496-id-1-2018_01_25_19_10_41.jfr|
| duration=time | 0s | JFRの記録時間を指定します。秒なら`s`, 分なら`m`, 時間なら`h`, 日なら`d`を付けます。|
| filename=path | - | JFRの記録終了時に書き込まれるファイル名を指定します。以下のように相対パスおよびWindows/Unix式でパスが指定できます。<br><ul><li>recording.jfr</li><li>c:\recordings\recording.jfr</li><li>/home/user/recordings/recording.jfr</li></ul>|
| name=identifier | - | 記録の名前と識別子を指定します|
| maxage=time | 0s | Diskにデータを保持する最大期間を指定します。指定を越えると古いものから消えていきます。秒なら`s`, 分なら`m`, 時間なら`h`, 日なら`d`を付けます。0の場合は無限に保存します。|
| maxsize=size | 0m | Diskにデータを保持する最大バイトサイズを指定します。指定を越えると古いものから消えていきます。単位としてm,M,g,Gを指定できます。0の場合は無限に保存します。|
| path-to-gc-roots={true, false} | false | GC Rootを診断記録に含めます。これはメモリリークの分析に有効ですがパフォーマンスペナルティがあるので疑わしい場合のみ有効にしてください。|
| settings=path | default.jfc | 記録に利用するイベント設定ファイル(JFCファイル)を指定します。デフォルトのディレクトリは`JRE_HOME/lib/jfr`です。`defualt.jfc`は必要最低限の値を取得するように設定してありオーバーヘッドが小さいです。必要に応じて`profile.jfc`やカスタムプロファイルに変更してください。|

### XX:FlightRecorderOptions

`-XX:FlightRecorderOptions=parameter=value`はjcmdの`JFR parameter`と等価なJFRオプションを指定するためのオプションです。
カンマ区切りで多くのオプションを指定することが出来ます。

| オプション| デフォルト値| 説明|
| -- | -- | -- |
| memorysize=size | 10MB | バッファメモリのサイズを指定します。この値はglobalbuffersizeとnumglobalbuffersのサイズのベースに利用されます。単位としてm,M,g,Gが指定できます。|
| globalbuffersize=size | - | グローバルバッファのメモリ量を指定します。デフォルトサイズはメモリサイズから自動で決まります。|
| numglobalbuffers=num | - | グローバルバッファのを指定します。デフォルト数はメモリサイズから自動できまります。|
| threadbuffersize=size | 8k | ローカルバッファサイズの指定。この値を変更するとオーバーヘッドが大きくなる恐れがある|
| maxchunksize=size | 12M | 最大チャンクサイズを指定します。単位としてm,M,g,Gが指定できます。最小サイズは1MBです。|
| old-object-queue-size=number-of-objects | 256 | キューイングする古いオブジェクトの最大数を指定します。|
| repository=path | システムの一時ディレクトリ(e.g. /tmp) | 循環バッファの値が書き込まれるディスク領域をディレクトリで指定します|
| retransform={true, false} | true | JVMTIを使用してイベントクラスを再変換するかどうかを指定します。 falseの場合、イベントクラスがロードされるときにInstrumentationが追加されます。|
| samplethreads={true, false} | true | スレッドサンプリングの取得の有無の指定|
| stackdepth=depth | 64 | サンプリングする素宅トレースの深さを指定。最大サイズは2048。64より大きな値を指定するとオーバーヘッドが非常に大きい|
| allow_threadbuffers_to_disk={true, false} | false | バッファスレッドがブロックされた場合に、バッファスレッドを直接ディスクに書くかを指定します。|

## プログラム上から記録する

JFR APIを使う事でプログラム上からJFRを記録することも可能です。

```java
var jfrConfig = jdk.jfr.Configuration.getConfiguration("default");
var path = Path.of("/tmp/myrecording.jfr");
try (var recording = new jdk.jfr.Recording(jfrConfig)) {
    recording.setName("Hello JFR");
    recording.start();

    // - ここに何か処理を入れる -

    recording.stop();
    recording.dump(path);
}
```

通常は利用することは少ないと思いますが、マイクロベンチマークの値をJFRに記録したい場合等には便利です。
