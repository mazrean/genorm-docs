# Install

GenORMはCLIにより生成したコードでテーブルのJoinなどを行い、クエリの発行は`genorm`パッケージを利用して行います。このため、CLIとPackage両方のinstallが必要です。

#### CLI

```
go install github.com/mazrean/genorm/cmd/genorm@v1.0.0
```

#### Package

```
go get -u github.com/mazrean/genorm
```
