---
layout: post
title: Protocol Buffers.md
categories: [其他]
description: 其他
keywords: 其他
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
# Protocol Buffers

## 简介

Protocol buffers缩写为protobuf，是由Google创造的一种用于序列化的标记语言，项目Github仓库：https://github.com/protocolbuffers/protobuf。

Protobuf主要用于不同的编程语言的协作RPC场景下，定义需要序列化的数据格式。Protobuf本质上仅仅是**一种用于交互的结构式定义**，从功能上**和XML、JSON**等各种其他的交互形式都**并无本质不同，只负责定义不负责数据编解码**。



### 多语言支持

protobuf是支持多种编程语言的，即多种编程语言的类型数据可以转换成protobuf定义的类型数据，各种语言的类型对应可以看此[介绍](https://developers.google.com/protocol-buffers/docs/proto3#scalar)。protobuf有个程序叫**protoc**，它是一个编译程序，**负责把proto文件编译成对应语言的文件**，它已经支持了C++、C#、Java、Python，而对于Go和Dart需要安装插件才能配合生成对于语言的文件。

对于Go，protoc需要使用插件**protoc-gen-go**，把`a.proto`，编译成`a.pb.go`，其中包含了定义的数据类型，它的序列化和反序列化函数等。protoc只负责利用protoc-gen-go把proto文件编译成Go语言文件，并不负责序列化和反序列化，Go语言对protobuf的序列化和反序列化是`github.com/golang/protobuf/proto`完成，它负责把结构体等序列化成proto数据(`[]byte`)，把proto数据反序列化成Go结构体。



## 语法

> request.proto

```protobuf
syntax = "proto3"; // 指定protobuf版本，现在是proto3
package helloworld; // 不完全等价于Go的package，最好另行设定go_package，指定根据protoc文件生成的go语言文件的package名称。
option go_package="./types";

message Request { // 会编译成Go的struct
    string data = 1;
}
```



### package 包

通过`package`关键字定义命名空间，用来隔离不同的消息类型的命名冲突。

```protobuf
package foo.bar;
message Open { ... }

//在另一个包中：
message Foo {
	required foo.bar.Open open = 1;
}
```



### option 选项

#### *_package

这里以`go_package`进行举例说明，该参数主要声明Go代码的存放位置。`package`参数针对的是protobuf，是proto文件的命名空间，它的作用是为了避免我们定义的接口，或者message出现冲突。

```protobuf
syntax = "proto3";

package demo;
option go_package="./demo";
```



### service 服务

通过`service`关键字定义服务：

```protobuf
service AdminUserService {
  rpc Detail(AdminUserDetailRequest) returns(AdminUserDetailResponse);
}
```



### message 消息



#### 字段修饰关键字

##### singular

`singular`是默认修饰符，可以省略。



##### repeated 

`repeated`表示字段类型为数据，映射到Java中为List，go中为slice。



#### 字段类型

##### 基础字段类型

一个标量消息字段可以含有一个如下的类型——该表格展示了定义于.proto文件中的类型，以及与之对应的、在自动生成的访问类中定义的类型：

| .proto Type | Notes                                                        | C++ Type | Java Type  | Python Type[2] | Go Type | Ruby Type                      | C# Type    | PHP Type       |
| ----------- | ------------------------------------------------------------ | -------- | ---------- | -------------- | ------- | ------------------------------ | ---------- | -------------- |
| double      |                                                              | double   | double     | float          | float64 | Float                          | double     | float          |
| float       |                                                              | float    | float      | float          | float32 | Float                          | float      | float          |
| int32       | 使用变长编码，对于负值的效率很低，如果你的域有可能有负值，请使用**sint32**替代 | int32    | int        | int            | int32   | Fixnum 或者 Bignum（根据需要） | int        | integer        |
| uint32      | 使用变长编码                                                 | uint32   | int        | int/long       | uint32  | Fixnum 或者 Bignum（根据需要） | uint       | integer        |
| uint64      | 使用变长编码                                                 | uint64   | long       | int/long       | uint64  | Bignum                         | ulong      | integer/string |
| sint32      | 使用变长编码，这些编码在负值时比int32高效的多                | int32    | int        | int            | int32   | Fixnum 或者 Bignum（根据需要） | int        | integer        |
| sint64      | 使用变长编码，有符号的整型值。编码时比通常的int64高效。      | int64    | long       | int/long       | int64   | Bignum                         | long       | integer/string |
| fixed32     | 总是4个字节，如果数值总是比总是比228大的话，这个类型会比uint32高效。 | uint32   | int        | int            | uint32  | Fixnum 或者 Bignum（根据需要） | uint       | integer        |
| fixed64     | 总是8个字节，如果数值总是比总是比256大的话，这个类型会比uint64高效。 | uint64   | long       | int/long       | uint64  | Bignum                         | ulong      | integer/string |
| sfixed32    | 总是4个字节                                                  | int32    | int        | int            | int32   | Fixnum 或者 Bignum（根据需要） | int        | integer        |
| sfixed64    | 总是8个字节                                                  | int64    | long       | int/long       | int64   | Bignum                         | long       | integer/string |
| bool        |                                                              | bool     | boolean    | bool           | bool    | TrueClass/FalseClass           | bool       | boolean        |
| string      | 一个字符串必须是UTF-8编码或者7-bit ASCII编码的文本。         | string   | String     | str/unicode    | string  | String (UTF-8)                 | string     | string         |
| bytes       | 可能包含任意顺序的字节数据。                                 | string   | ByteString | str            | []byte  | String (ASCII-8BIT)            | ByteString | string         |



##### 消息字段类型

在消息中使用其他消息作为字段类型：

```protobuf
message SearchResponse {
	repeated Result results = 1;  //使用了下面的Result作为类型
}

message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
}
```



如果是使用的其他消息内部定义的消息作为字段类型，则需要指明父级：

```protobuf
message SearchResponse {
    message Result {     //在消息SearchResponse定义消息Result
        string url = 1;
        string title = 2;
        repeated string snippets = 3;
    }
    repeated Result results = 1;    //在消息SearchResponse内重用上面的Result定义作为类型
}

//外部重用
message SomeOtherMessage {
	SearchResponse.Result result = 1;  //需要指明父级
}
```



##### map字段类型

可以通过`map`关键字定义自字段为map类型数据，`map`的值不可为`repeated`的。

```protobuf
message Request {
  string ping = 1;
  map<string, string> header = 2;
}
```



### enum 枚举

可以通过使用`enum`关键字定义枚举，因为`enum`值是使用可变编码方式的，对负数不够高效，因此不推荐在`enum`中使用负数。

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;  //Corpus枚举类型，字段名是corpus， 标号为4(不是枚举中的值，是标号)
}
```



在定义枚举时通过设置`option allow_alias = true;`来表示枚举值允许重复：

```protobuf
enum EnumAllowingAlias {
    option allow_alias = true; //开启这个选项才能允许重复
    UNKNOWN = 0;
    STARTED = 1;
    RUNNING = 1;  //与上面的STARTED重复
}

enum EnumNotAllowingAlias {
    UNKNOWN = 0;
    STARTED = 1;
    // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```



### import 导入

可以使用`import`关键字导入其他.proto文件来实现重用的效果。

```protobuf
import "myproject/other_protos.proto";

// 另外还有一种import public的东西，看图说话：
// new.proto
// All definitions are moved here
 
// old.proto
// This is the proto that all clients are importing.
import public "new.proto"; //伪文件,导入old.proto的时候，会导入new.proto,依赖也会传递下去。用途类似unix的符号链接
import "other.proto";
 
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, *** but not other.proto ***
```



## 命令

### 命令参数

#### --help

`--help` 查看protoc命令帮助信息。

```sh
$ protoc --help
Usage: D:\GitProject\go\bin\protoc.exe [OPTION] PROTO_FILES
Parse PROTO_FILES and generate output based on the options given:
  -IPATH, --proto_path=PATH   Specify the directory in which to search for
                              imports.  May be specified multiple times;
                              directories will be searched in order.  If not
                              given, the current working directory is used.
                              If not found in any of the these directories,
                              the --descriptor_set_in descriptors will be
                              checked for required proto file.
  --version                   Show version info and exit.
  -h, --help                  Show this text and exit.
  --encode=MESSAGE_TYPE       Read a text-format message of the given type
                              from standard input and write it in binary
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --deterministic_output      When using --encode, ensure map fields are
                              deterministically ordered. Note that this order
                              is not canonical, and changes across builds or
                              releases of protoc.
  --decode=MESSAGE_TYPE       Read a binary message of the given type from
                              standard input and write it in text format
                              to standard output.  The message type must
                              be defined in PROTO_FILES or their imports.
  --decode_raw                Read an arbitrary protocol message from
                              standard input and write the raw tag/value
                              pairs in text format to standard output.  No
                              PROTO_FILES should be given when using this
                              flag.
  --descriptor_set_in=FILES   Specifies a delimited list of FILES
                              each containing a FileDescriptorSet (a
                              protocol buffer defined in descriptor.proto).
                              The FileDescriptor for each of the PROTO_FILES
                              provided will be loaded from these
                              FileDescriptorSets. If a FileDescriptor
                              appears multiple times, the first occurrence
                              will be used.
  -oFILE,                     Writes a FileDescriptorSet (a protocol buffer,
    --descriptor_set_out=FILE defined in descriptor.proto) containing all of
                              the input files to FILE.
  --include_imports           When using --descriptor_set_out, also include
                              all dependencies of the input files in the
                              set, so that the set is self-contained.
  --include_source_info       When using --descriptor_set_out, do not strip
                              SourceCodeInfo from the FileDescriptorProto.
                              This results in vastly larger descriptors that
                              include information about the original
                              location of each decl in the source file as
                              well as surrounding comments.
  --dependency_out=FILE       Write a dependency output file in the format
                              expected by make. This writes the transitive
                              set of input file paths to FILE
  --error_format=FORMAT       Set the format in which to print errors.
                              FORMAT may be 'gcc' (the default) or 'msvs'
                              (Microsoft Visual Studio format).
  --fatal_warnings            Make warnings be fatal (similar to -Werr in
                              gcc). This flag will make protoc return
                              with a non-zero exit code if any warnings
                              are generated.
  --print_free_field_numbers  Print the free field numbers of the messages
                              defined in the given proto files. Groups share
                              the same field number space with the parent
                              message. Extension ranges are counted as
                              occupied fields numbers.
  --plugin=EXECUTABLE         Specifies a plugin executable to use.
                              Normally, protoc searches the PATH for
                              plugins, but you may specify additional
                              executables not in the path using this flag.
                              Additionally, EXECUTABLE may be of the form
                              NAME=PATH, in which case the given plugin name
                              is mapped to the given executable even if
                              the executable's own name differs.
  --cpp_out=OUT_DIR           Generate C++ header and source.
  --csharp_out=OUT_DIR        Generate C# source file.
  --java_out=OUT_DIR          Generate Java source file.
  --js_out=OUT_DIR            Generate JavaScript source.
  --kotlin_out=OUT_DIR        Generate Kotlin file.
  --objc_out=OUT_DIR          Generate Objective-C header and source.
  --php_out=OUT_DIR           Generate PHP source file.
  --python_out=OUT_DIR        Generate Python source file.
  --ruby_out=OUT_DIR          Generate Ruby source file.
  @<filename>                 Read options and filenames from file. If a
                              relative file path is specified, the file
                              will be searched in the working directory.
                              The --proto_path option will not affect how
                              this argument file is searched. Content of
                              the file will be expanded in the position of
                              @<filename> as in the argument list. Note
                              that shell expansion is not applied to the
                              content of the file (i.e., you cannot use
                              quotes, wildcards, escapes, commands, etc.).
                              Each line corresponds to a single argument,
                              even if it contains spaces.
```



#### --*_out

语言参数即上述的`--cpp_out=`，`--python_out=`等，protoc支持的语言长达13种，且都是比较常见的。像上面出现的语言参数，说明protoc本身已经内置该语言对应的编译插件，无需安装。

而如果上面没出现的，比如`--go_out=`，就得自己单独安装语言插件，比如`--go_out=`对应的是`protoc-gen-go`，安装命令如下：

```shell
# 最新版
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest

# 指定版本
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.3.0
```

如果想了解更多语言的编译插件，可点击[protobuf官方文档](https://link.juejin.cn?target=https%3A%2F%2Fdevelopers.google.com%2Fprotocol-buffers%2Fdocs%2Freference%2Foverview)进行查阅。



#### --go_out

`--go_out`指明了要把`./request.proto`编译成Go语言文件。

```sh
$ protoc --go_out=. ./request.proto  # -–go_out=后直接跟路径(点号表示当前路径)。只生成序列化/反序列化代码文件，不需要rpc通讯。
$ protoc --go_out=plugins=grpc:.  ./request.proto     # --go_out=后跟了grpc插件和目录。生成序列化反序列化代码和客户端/服务端通讯代码。
```



#### --proto_path

`--proto_path` 指定查找.proto文件的路径，也可以使用`-I`和`-IPATH`指定，注意使用`-I`时不需要`=`拼接文件路径。

```sh
$ protoc --proto_path=. --go_out=. ./request.proto
$ protoc -I. --go_out=. ./request.proto
```



`--go_out`主要的两个参数为`plugins` 和 `paths`，分别表示生成Go代码所使用的插件，以及生成的Go代码的位置。`--go_out`的写法是，参数之间用逗号隔开，最后加上冒号来指定代码的生成位置，比如`--go_out=plugins=grpc,paths=import:.`。

`plugins`参数有不带grpc和带grpc两种（应该还有其它的，目前知道的有这两种），两者的区别如下，带grpc的会多一些跟gRPC相关的代码，实现gRPC通信。`paths`参数有两个选项，分别是 `import` 和 `source_relative`，默认为 import，表示按照生成的Go代码的包的全路径去创建目录层级，source_relative 表示按照 **proto源文件的目录层级**去创建Go代码的目录层级，如果目录已存在则不用创建。

```protobuf
option go_package="proto1/pb_go";
```

```sh
# 指令1：paths为import，pb文件最终在 pb_go 目录下
$ protoc --proto_path=. --go_out=. proto1/greeter/greeter_v2.proto
$ protoc --proto_path=. --go_out=paths=import:. proto1/greeter/greeter_v2.proto

# 指令2：paths为source_relative，pb文件最终在 proto1/greeter 目录下
$ protoc --proto_path=. --go_out=paths=source_relative:. proto1/greeter/greeter_v2.proto
```



#### --version

`--version`查看protoc版本。

```sh
$ protoc --version
libprotoc 3.19.4
```



#### @\<filename>

proto文件位置参数即上述的`@<filename>`参数，指定了.proto文件的具体位置，如`./request.proto`。



## 高级应用

### 变更升级

**升级建议：**

- 不要更改任何已有的字段的数值标识。
- 如果你增加新的字段，使用旧格式的字段仍然可以被你新产生的代码所解析。你应该记住这些元素的默认值这样你的新代码就可以以适当的方式和旧代码产生的数据交互。相似的，通过新代码产生的消息也可以被旧代码解析：只不过新的字段会被忽视掉。注意，未被识别的字段会在反序列化的过程中丢弃掉，所以如果消息再被传递给新的代码，新的字段依然是不可用的（这和proto2中的行为是不同的，在proto2中未定义的域依然会随着消息被序列化）
- 非required的字段可以移除——只要它们的标识号在新的消息类型中不再使用（更好的做法可能是重命名那个字段，例如在字段前添加“OBSOLETE_”前缀，那样的话，使用的.proto文件的用户将来就不会无意中重新使用了那些不该使用的标识号）。
- int32, uint32, int64, uint64,和bool是全部兼容的，这意味着可以将这些类型中的一个转换为另外一个，而不会破坏向前、 向后的兼容性。如果解析出来的数字与对应的类型不相符，那么结果就像在C++中对它进行了强制类型转换一样（例如，如果把一个64位数字当作int32来 读取，那么它就会被截断为32位的数字）。
- sint32和sint64是互相兼容的，但是它们与其他整数类型不兼容。
- string和bytes是兼容的——只要bytes是有效的UTF-8编码。
- 嵌套消息与bytes是兼容的——只要bytes包含该消息的一个编码过的版本。
- fixed32与sfixed32是兼容的，fixed64与sfixed64是兼容的。
- 枚举类型与int32，uint32，int64和uint64相兼容（注意如果值不相兼容则会被截断），然而在客户端反序列化之后他们可能会有不同的处理方式，例如，未识别的proto3枚举类型会被保留在消息中，但是他的表示方式会依照语言而定。int类型的字段总会保留他们的



## 参考资料

- [Go是如何实现protobuf的编解码的(1)：原理](https://lessisbetter.site/2019/08/26/protobuf-in-go/)
