# Code Generation

Configurationファイルを書いたら`genorm`コマンドを用いてコード生成します。`genorm`コマンドのflagの概要は以下の通りです。

```
$ genorm -help
Usage of ./genorm:
  -destination string
    	The destination file to write.
  -join-num int
    	The number of joins to generate. (default 5)
  -module string
    	The root module name to use.
  -package string
    	The root package name to use.
  -source string
    	The source file to parse.
  -version
    	If true, output version information.
```

Configurationファイルなどに`//go:generate`ディレクティブを書くことで`go generate`によりコード生成できます。

### destination(required)

コードを生成する対象のディレクトリです。コマンドを実行しているディレクトリからの相対パス、絶対パスで指定できます。手で書くコードとは異なるディレクトリを指定することをお勧めします。

### module(required)

コード生成先ディレクトリのmodule名を指定します。

例えば、以下のようなディレクトリ構成でgo.modで指定されたmodule名が`github.com/mazrean/genorm-workspace`である場合は`github.com/mazrean/genorm-workspace/genorm`となります。

```
.
├── genorm //生成先ディレクトリ
└── go.mod
```

### package(required)

コード生成先ディレクトリ直下のpackage名を指定します。

例えば、`destination`flagで`./genorm`を指定し`package`flagにormを設定したとき、`./genorm`直下に生成される`genorm.go`ファイルの先頭に`package orm`が書き込まれ、`./genorm`直下が`orm`packageとなります。

### source(required)

Configurationファイルのパスです。コマンドを実行しているディレクトリからの相対パス、絶対パスで指定できます。

### join-num

Join可能なテーブル数を指定します。デフォルト値は5です。

{% hint style="info" %}
コード生成時間はこの値に対して指数的に増加します。大きくする際にはコード生成にかかる時間に注意してください。また、コード生成時間が長い場合はこの値を小さくすることを検討してください。
{% endhint %}

### version

コマンドのバージョンを出力します。
