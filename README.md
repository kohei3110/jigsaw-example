## Jigsawとは
 - Project Jigsawという、Javaのプラットフォームをモジュールに分割するためのプロジェクト
 - Java SE 9から、JDKがモジュールに分割され、モジュールを使ったアプリケーションの設計が可能になった

## モジュールとは
 - カプセル化されたソースコードの集まり
 - 他のモジュールとの依存関係や公開範囲を明確化できる

## モジュール化によるメリット
 - 外部に公開する or しないを明確化可能
 - クラス同士の予期せぬ依存関係を防止
 - 実行時にクラスパスが存在しないことに起因するエラーを防止
 - JVM起動時に必要なモジュールを最適化可能

## クラスパスとモジュールパス
 - Java SE 8までは、クラスパスを参照してJVMがクラスをロードしていた
 - Java SE 9からは、クラスロード時にモジュールパスが使用される（クラスパスは存在するが、モジュールパスからクラスパスを参照することは不可能）

## 【補足】名前付きモジュール(Named Module) / 自動モジュール / 無名モジュール
Java 9以降のモジュールシステムでは、モジュールの種類を「名前付きモジュール」「自動モジュール」「無名モジュール」の3つに分類できる。

### 名前付きモジュール(Named Module)
 - Java 9で導入されたモジュールシステムに対応して定義されたモジュールは、自動的に名前付きモジュールとなる
 - module-info.javaによってモジュール定義が行われたモジュールを指す

### 自動モジュール(Automatic Module)
 - module-info.java によるモジュール定義を持たない形式のJarファイルがモジュールパス上に配置された場合、自動的にモジュールとして扱われる
 - jarファイル内で定義されたパッケージは自動的に全てexportsされたものとして扱われる

### 無名モジュール(Unnamed Module)
 - クラスパスに配置されたJarファイルのクラスやパッケージは、無名モジュールと呼ばれるモジュールに所属する
 - ほかのモジュールからモジュール名を指定して明示的に依存関係を定義することは不可能
 - 代わりに、無名モジュール内のすべてのパッケージは自動的にすべてexportsされたものとして扱われる
 - モジュールグラフに読み込まれたすべてのモジュールをrequiresしているものとして扱われる

## コンパイル / 実行方法

### 前提
```
$ java -version
java version "11.0.1" 2018-10-16 LTS
Java(TM) SE Runtime Environment 18.9 (build 11.0.1+13-LTS)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11.0.1+13-LTS, mixed mode)
```
### コンパイル
```
（使用法）
$ javac -d <クラスファイルの出力先> <コンパイル対象ファイル>

（例）
$ javac -d mods/com.kohsaito src/com.kohsaito/module-info.java src/com.kohsaito/com/kohsaito/World.java

$ javac --module-path mods -d mods/com.greetings/ src/com.greetings/module-info.java src/com.greetings/com/greetings/Main.java
```

複数モジュールを同時にコンパイルすることも可能
```
$ javac -d mods --module-source-path src $(find src -name "*.java")
```

### 実行
```
（使用法）
$ java --module-path <モジュールの検索先> -m <モジュール名>/<クラス名>

（例）
$ java --module-path mods -m com.greetings/com.greetings.Main

My name is : kohei
```

## jarファイル(モジュラjarファイル)にパッケージング
モジュールをjarファイルとしてまとめておくと、デプロイ時に便利。<br>
※モジュラjarファイル：ルートディレクトリにモジュール記述子module-info.classを持つJARファイル。モジュール記述子は、モジュール宣言のバイナリ形式である。

```
$ mkdir mlib

$ jar --create --file=mlib/com.kohsaito@1.0.jar --module-version=1.0 -C mods/com.kohsaito .

$ jar --create --file=mlib/com.greetings.jar --main-class=com.greetings.Main -C mods/com.greetings .
```

## モジュラjarファイルにパッケージされているモジュール一覧を出力
```
$ jar --describe-module --file=mlib/com.kohsaito\@1.0.jar 
com.kohsaito@1.0 jar:file:///Users/kohei/work/module/greetings/mlib/com.kohsaito@1.0.jar/!module-info.class
exports com.kohsaito
requires java.base mandated
```
