# 目次

## はじめに

## 1章　JDK Flight Recoder とは何か

- 1.1 JDK Flight Recorderとは？
- 1.2 Javaにおけるパフォーマンス分析と障害診断
- 1.3 JFR の歴史 - JRockitからOpenJDKまで
- 1.4 JFRを取り巻くツール(jcmd, JFR tool, JDK Mission Control)
- 1.5 JFRとJMCのインストール

## 2章　JDK Flight Recorderのアーキテクチャ

- 2.1 

## 3章　JDK Flight Recorderの実行

- 3.1 JMCから利用する
- 3.2 jcmdの使用
- 3.3 コマンドラインオプション
- 3.4 ..

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

## 6章　JDK Flight Record API

- 6.1 カスタムイベント
- 6.2 JFRのコントロール
- 6.3 JFR Consuming API
- 6.4 カスタムイベントにるSQLのロギング
- 6.x JFRとOpenTracingの連携

## 7章　モニタリングへの応用

- 7.1 JFRのモニタリングへの課題
- 7.2 JFR Consuming APIとJFR toolの活用
- 7.3 KiabanaによるJFRのモニタリング
- 7.4 ...

## x章　その他

- x.x JRubyとJFR
- x.x GlaalVMやOpenJ9におけるJFR
- x.x ロギングツールとしてのJFR
