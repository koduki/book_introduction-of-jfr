# 3.1 JMCから利用する

## JFRの取得

JFRを取得するには以下のいずれかの方法でJFRの記録を開始する必要があります。

1. GUI(JMC/VisualVM)で記録する
2. CLIで記録する
3. JVMオプションで記録する
4. プログラム上から記録する

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
| disk={true|false} |false | 循環バッファの値をディスクに書き込むかどうかを指定します。|


dumponexit={true|false}
Specifies if the running recording is dumped when the JVM shuts down. If enabled and a filename is not entered, the recording is written to a file in the directory where the process was started. The file name is a system-generated name that contains the process ID, recording ID, and current timestamp, similar to hotspot-pid-47496-id-1-2018_01_25_19_10_41.jfr. By default, this parameter is disabled.

duration=time
Specifies the duration of the recording. Append s to specify the time in seconds, m for minutes, h for hours, and d for days. For example, specifying 5h means 5 hours. By default, the duration isn’t limited, and this parameter is set to 0.

filename=path
Specifies the path and name of the file to which the recording is written when the recording is stopped, for example:

recording.jfr
/home/user/recordings/recording.jfr
c:\recordings\recording.jfr
name=identifier
Takes both the name and the identifier of a recording.

maxage=time
Specifies the maximum age of disk data to keep for the recording. This parameter is valid only when the disk parameter is set to true. Append s to specify the time in seconds, m for minutes, h for hours, and d for days. For example, specifying 30s means 30 seconds. By default, the maximum age isn’t limited, and this parameter is set to 0s.

maxsize=size
Specifies the maximum size (in bytes) of disk data to keep for the recording. This parameter is valid only when the disk parameter is set to true. The value must not be less than the value for the maxchunksize parameter set with -XX:FlightRecorderOptions. Append m or M to specify the size in megabytes, and g or G to specify the size in gigabytes. By default, the maximum size of disk data isn’t limited, and this parameter is set to 0.

path-to-gc-roots={true|false}
Specifies whether to collect the path to garbage collection (GC) roots at the end of a recording. By default, this parameter is disabled.

The path to GC roots is useful for finding memory leaks, but collecting it is time-consuming. Enable this option only when you start a recording for an application that you suspect has a memory leak. If the settings parameter is set to profile, the stack trace from where the potential leaking object was allocated is included in the information collected.

settings=path
Specifies the path and name of the event settings file (of type JFC). By default, the default.jfc file is used, which is located in JRE_HOME/lib/jfr. This default settings file collects a predefined set of information with low overhead, so it has minimal impact on performance and can be used with recordings that run continuously.

A second settings file is also provided, profile.jfc, which provides more data than the default configuration, but can have more overhead and impact performance. Use this configuration for short periods of time when more information is needed.

You can specify values for multiple parameters by separating them with a comma.

