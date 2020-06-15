# Flight Recorder APIとは？

## Flight Recorder API

JFRではJDK9より以前にも`com.oracle.jrockit.jfr`という形でJFRをJava側から操作したりカスタムイベントを作る機能がありました。
しかし、これはJRockitから引き継いだ非公式APIのため、ドキュメントも基本的にありませんしいつ廃止になるかも分からない状態でした。

そのため、あまり大っぴらに使われる機能ではなくWeblogicなどのOracle製品に使われてていたり、エンジニアが社内で内部的に使うに留まっていたのですが、JDK9より`jdk.jfr` APIとして刷新され公式サポートされたことにより状況は大きく変わりました。

もっとも大きな変化としては、公式APIなのでドキュメントに記載されるようになったことです。

- Flight Recorder APIプログラマーズ・ガイド: https://docs.oracle.com/javase/jp/14/jfapi/preface.html
- Javadoc: https://docs.oracle.com/en/java/javase/14/docs/api/jdk.jfr/jdk/jfr/package-summary.html

当たり前の言えば当たり前なのですが、これによって **秘伝のタレ** 化していたAPIの使い方が一般化されたのが非常に大きいです。これまではOracle社のコンサルに聞いたり公開されているスライドやTips集から情報を収集し、しかもそれが今も正しいかが分からない手探りの中の運用という状態だったので大きな前進となります。

## Flight Recoder APIの構成

JFR APIは大別すると以下の２つに分けられます。

- Custom Event
- Consumer/Recording API

Custom Eventは独自のイベントを作ってJFRの分析機能を強化する機能です。
たとえば、JFRの標準ではCPU, メモリ, GC, I/O, Method Profileなどの基礎的な値は取得しますが、WebアプリケーションのレスポンスやSQLの実行時間などミドルウェアやアプリケーションのメトリクスを取得しません。
ただ、WeblogicなどOracle製品ではSQLの情報やJAX-RSのレスポンス値等のミドルウェア固有の値も取得することができます。カスタムイベントを利用することで、GlassFishやQuarkusといったOSSのミドルウェア情報も取得できるだけではなく、アプリケーション自身の固有のライブラリやロジックを計測できるようになります。

Consumer/Recording APIはJFRファイルを操作するためのAPIです。JFRの記録の開始をプログラム側で制御したり、バイナリであるJFRファイルを解析する時に使います。特にConsume APIはJFRのデータを直接解析したりJSONやXMLなど任意のフォーマットに加工したりできるのでとても有用です。JFRコマンドで代用できる部分も多いですが、よりきめ細やかな操作をしたい場合にはこちらが使えるでしょう。
また、JDK14よりJEP 349としてJFR Event Streamingが追加されました。Event Streamingを使うことでJFRでイベントが記録されたタイミングでアクションを起こせるので、一部データのログファイルへの出力や既存のモニタリングシステム/分析ツールにJFRの情報をリアルタイムに連携できるようになりました。

次章以降では具体的なサンプルを混ぜながらそれぞれの詳細に関して説明していきたいと思います。


