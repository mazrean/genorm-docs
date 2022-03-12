# Code Generation

After writing the Configuration file, run the `genorm` command to generate the code. The following is an overview of the flags in the `genorm` command.

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

Code can be generated by `go generate` by writing `//go:generate` directives in configuration files, etc.

### destination(required)

The directory in which the code is to be generated. It can be specified as a relative or absolute path from the directory where the command is executed. It is recommended that you specify a different directory than the code you write by hand.

### module(required)

Specify the module name of the directory in which the code is to be generated.

For example, if the module name specified in `go.mod` is `github.com/mazrean/genorm-workspace` in the following directory structure, then `github.com/mazrean/genorm-workspace/genorm` and The following is a list of the most common problems that occur in the system.

```
.
├── genorm //destination directory
└── go.mod
```

### package(required)

Specify the name of the package directly under the directory where the code will be generated.

For example, if the `destination` flag is `. /genorm` in the `destination` flag and set `orm` in the `package` flag. /genorm`, the `package orm` will be written at the top of the `genorm.go` file generated directly under `. /genorm` becomes `orm`package.

### source(required)

The path to the Configuration file. It can be specified as a relative or absolute path from the directory where the command is executed.

### join-num

Specifies the number of tables that can be joined. The default value is 5.

{% hint style="info" %}
Code generation time increases exponentially with this value. Note the time required for code generation when increasing the value. Also, if the code generation time is long, consider decreasing this value.
{% endhint %}

### version

If set, outputs the version of the command.