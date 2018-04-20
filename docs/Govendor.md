# 依赖管理

文档维护 | 辰枫
---|---
更新日期 | 2017-11-24
文档版本 | v1.0


### govendor常用命令

>`govendor init` #初始化所有依赖

>`govendor fetch +out` #安装所有依赖

>`govendor test +local` #使用本地依赖运行单元测试

>`govendor build`


长期以来，golang 对外部依赖都没有很好的管理方式，只能从 $GOPATH 下查找依赖。这就造成不同用户在安装同一个项目适合可能从外部获取到不同的依赖库版本，同时当无法联网时，无法编译依赖缺失的项目。

自 1.5 版本开始引入 govendor 工具，该工具将项目依赖的外部包放到项目下的 vendor 目录下（对比 nodejs 的 node_modules 目录），并通过 vendor.json 文件来记录依赖包的版本，方便用户使用相对稳定的依赖。

对于 govendor 来说，主要存在三种位置的包：项目自身的包组织为本地（local）包；传统的存放在 $GOPATH 下的依赖包为外部（external）依赖包；被 govendor 管理的放在 vendor 目录下的依赖包则为 vendor 包。

具体来看，这些包可能的类型如下：

状态|缩写状态|含义
---|---|---
+local|	l	|本地包，即项目自身的包组织
+external|	e	|外部包，即被 $GOPATH 管理，但不在 vendor 目录下
+vendor	|v	|已被 govendor 管理，即在 vendor 目录下
+std	|s	|标准库中的包
+unused	|u	|未使用的包，即包在 vendor 目录下，但项目并没有用到
+missing|	m|	代码引用了依赖包，但该包并没有找到
+program|	p|	主程序包，意味着可以编译为执行文件
+outside|	 |	外部包和缺失的包
+all|	|所有的包

常见的命令如下，格式为 govendor COMMAND。

通过指定包类型，可以过滤仅对指定包进行操作。


参数|解释
---|---
init    |   创建 vendor 文件夹和 vendor.json 文件
list    |   列出已经存在的依赖包
add     |   从 $GOPATH 中添加依赖包，会加到 vendor.json
update  |   从 $GOPATH 升级依赖包
remove  |   从 vendor 文件夹删除依赖
status  |   列出本地丢失的、过期的和修改的package
fetch   |   从远端库增加新的，或者更新 vendor 文件中的依赖包
sync    |   本地存在 vendor.json 时候拉取依赖包，匹配所记录的版本
migrate |   Move packages from a legacy tool to the vendor folder with metadata.
get     |   类似 go get，但是会把依赖包拷贝到 vendor 目录
license |   List discovered licenses for the given status or import paths.
shell   |   Run a "shell" to make multiple sub-commands more efficient for large projects.

> go tool commands that are wrapped:
> 	  `+<status>` package selection may be used with them
> 	fmt, build, install, clean, test, vet, generate, tool



