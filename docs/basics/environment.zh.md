# 环境

Vapor 的环境API帮助您动态配置您的应用程序。默认情况下，你的应用程序将使用 `development` 环境。你可以定义其他有用的环境，如 `production` 或 `staging`，并在每种情况下改变你的应用是如何配置的。您还可以从进程的环境或 `.Env` (dotenv)文件读取配置取决于您的需要。

要访问当前环境，请使用 `app.environment`。你可以在 `configure(_:)` 中通过这个属性来执行不同的配置逻辑。
To access the current environment, use `app.environment`. You can switch on this property in `configure(_:)` to execute different configuration logic. 

```swift
switch app.environment {
case .production:
    app.databases.use(....)
default:
    app.databases.use(...)
}
```

## 改变环境

默认情况下，你的应用程序将在 `development` 环境中运行。你可以通过在应用程序引导期间传递`--env` (`-e`)标志来改变这一点。

```swift
vapor run serve --env production
```

Vapor 包含下列环境：

|name|short|description|
|-|-|-|
|production|prod|Deployed to your users.|
|development|dev|Local development.|
|testing|test|For unit testing.|

!!! info "信息"
    `production` 环境将默认为 `notice` 级别的日志记录，除非另有说明。所有其他环境默认为 `info`。

您可以将全名或短名传递给`--env` (`-e`)标志。

```swift
vapor run serve -e prod
```

## 进程变量

`Environment` 提供了一个简单的、基于字符串的API来访问进程的环境变量。

```swift
let foo = Environment.get("FOO")
print(foo) // String?
```

除了 `get` 之外，`Environment` 还通过 `process` 提供了一个动态成员查找API。

```swift
let foo = Environment.process.FOO
print(foo) // String?
```

当在终端运行应用程序时，你可以使用 `export` 设置环境变量。

```sh
export FOO=BAR
vapor run serve
```

当在Xcode中运行应用程序时，你可以通过编辑 `Run` scheme来设置环境变量。

## .env (dotenv)

Dotenv文件包含一个键值对列表，这些键值对将自动加载到环境中。这些文件使配置环境变量变得很容易，而不需要手动设置它们。

Vapor 将在当前工作目录中查找dotenv文件。如果你使用Xcode，确保通过编辑 `Run` scheme 设置工作目录。

Assume the following `.env` file placed in your projects root folder:
假设以下 `.env` 文件放在你的项目根文件夹中:

```sh
FOO=BAR
```

当您的应用程序启动时，您将能够像访问其他进程环境变量一样访问该文件的内容。

```swift
let foo = Environment.get("FOO")
print(foo) // String?
```

!!! info "信息"
    在 `.env` 文件中指定的变量不会覆盖进程环境中已经存在的变量。

在`.env`旁边，Vapor 还将尝试为当前环境加载一个dotenv文件。例如，在 `development` 环境中，蒸汽将加载 `.env.development`。特定环境文件中的任何值都将优先于 `.env` 文件内的值。

一个典型的模式是项目包含一个 `.env` 文件作为带有默认值的模板。在 `.gitignore` 中使用以下模式忽略特定的环境文件

```gitignore
.env.*
```

当项目被 cloned 到新计算机时，已经带有正确的值的`.env`模板可以被复制。

```sh
cp .env .env.development
vim .env.development
```

!!! warning "警告"
    带有敏感信息(如密码)的Dotenv文件不应提交给版本控制。

如果你在加载dotenv文件时遇到了困难，尝试使用 `--log debug` 来启用调试日志以获取更多信息。

## 自定义环境

要定义自定义的环境名称，请扩展 `Environment`。

```swift
extension Environment {
    static var staging: Environment {
        .custom(name: "staging")
    }
}
```

应用程序的环境通常使用 `main.swift` 中的 `environment .detect()` 来设置。

```swift
import Vapor

var env = try Environment.detect()
try LoggingSystem.bootstrap(from: &env)

let app = Application(env)
defer { app.shutdown() }
```

`detect` 方法使用进程的命令行参数并自动解析 `--env`标志。您可以通过初始化自定义的 `Environment` 结构来覆盖此行为。

```swift
let env = Environment(name: "testing", arguments: ["vapor"])
```

参数数组必须包含至少一个表示可执行名称的参数。可以提供进一步的参数来模拟通过命令行传递参数。这对于测试特别有用。
