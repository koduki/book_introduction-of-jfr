# 目次

## [はじめに](preface.md)

## 1章　JDK Flight Recoderとは？

- [1.1 JDK Flight Recorder](01/01-what_is_JFR.md)
- [1.2 Javaにおけるパフォーマンス分析と障害診断](01/02-other_tools.md)
- [1.3 JFRの歴史 - JRockitからOpenJDKまで](01/03-history_of_jfr.md)
- [1.4 JFRの動作環境とJMCのインストール](01/04-install_jmc.md)

## 2章　JDK Flight Recorderのアーキテクチャ

- [2.1 JFRのアーキテクチャ概要](02/01-jfr-architecture.md)
- 2.2 JFRとオーバーヘッド

## 3章　JDK Flight Recorderの記録

- [3.1 JDK Flight Recorderの記録](03/01-recording-jfr.md)

## 4章　JDK Mission Controlによる障害分析

- 4.1 JFR/JMCで分析可能なメトリクス
- 4.2 Weblogic(WLDF)とJFR
- 4.3 ECIDと分散トレース
- 4.x ユースケース
    - 4.x.1 ユースケース1 - バッチのボトルネック分析
    - 4.x.2 ユースケース2 - WebアプリケーションでのフルGC発生の調査

## 5章　本番環境への適用とログローテーション

- 5.1 ...
- 5.x 巨大なJFRファイルの分割
- 5.x リポジトリから毎時/日次のログを作る

## 6章 Flight Record API
- [6.1 Flight Recorder APIとは？](06/01-what_is_JFR_api.md)
- [6.2 カスタムイベント](06/02-custom_event.md)
- 6.3 JFRのコントロール
- 6.4 JFR Consuming API
- 6.5 JFR Event Streaming API
- 6.6 カスタムイベントにるSQLのロギング
- 6.7 JFRとOpenTracingの連携

## 7章　モニタリングへの応用

- 7.1 JFRのモニタリングへの課題
- 7.2 JFR Consuming APIとJFR toolの活用
- 7.3 KiabanaによるJFRのモニタリング
- 7.4 ...

## x章　その他

- x.x JRubyとJFR
- x.x GlaalVMやOpenJ9におけるJFR
- x.x ロギングツールとしてのJFR
