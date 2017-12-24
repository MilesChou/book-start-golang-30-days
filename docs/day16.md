# Dep

學一個程式語言，最快的學法就是直接從實作中學。從今天開始要進入應用程式實作階段了！

最終想做的應用程式為：[姓名產生器](https://github.com/MilesChou/namer)。

簡單介紹：透過應用程式可以產生姓名，並可以建立屬於自己的字典，讓取名可以有自定義的方向，且可避開不喜歡的字。

---

接下來的方式會是一天一個小迭代，過程中會盡量引發問題，才有機會學到更多東西！

迭代的方法為：

1. 設定當天目標
2. 分析
3. 開工
4. 如果有發生問題，就解決遇到的問題
5. 展示

## 今日目標

使用 [dep][] 初始化一個 CLI 專案。

## 分析

首先，在那之前，你要先有 dep ：

```bash
$ brew install dep
```

再來找現有程式參考。 [Dapper][] 是一個不錯且夠簡單的 CLI 專案，它的 `main.go` 裡面用了 [urfave/cli][] （有改名過）框架，今天就來試著建置起來吧。

## 開工

首先應該先下載 cli 套件原始碼：

```bash
$ go get github.com/urfave/cli
```

建立主要檔案 `main.go`

```go
package main

import (
	"os"
	"github.com/urfave/cli"
)

func main() {
	app := cli.NewApp()
	app.Name = `Namer`
	app.Run(os.Args)
}
```

接著來了解 dep 該怎麼用

```bash
$ dep
dep is a tool for managing dependencies for Go projects

Usage: dep <command>

Commands:

  init     Initialize a new project with manifest and lock files
  status   Report the status of the project's dependencies
  ensure   Ensure a dependency is safely vendored in the project
  prune    Prune the vendor tree of unused packages
  version  Show the dep version information

Examples:
  dep init                               set up a new project
  dep ensure                             install the project's dependencies
  dep ensure -update                     update the locked versions of all dependencies
  dep ensure -add github.com/pkg/errors  add a dependency to the project

Use "dep help [command]" for more information about a command.
```

看起來可以先使用 `init` 

```bash
$ dep init
  Using ^1.20.0 as constraint for direct dep github.com/urfave/cli
  Locking in v1.20.0 (cfb3883) for direct dep github.com/urfave/cli
```

它也很聰明，幫我把程式碼裡面的依賴都找一輪，然後也下載到 `vendor` 目錄裡。

## 問題

`vendor` 要不要 commit ，可以參考[官方文件](https://github.com/golang/dep/blob/master/docs/FAQ.md#should-i-commit-my-vendor-directory)，它是說隨意，同時也很負責的說明了優缺點。而筆者決定 commit 。

## 展示

執行後就能看到最簡單的 command 了：

```bash
$ go run main.go
NAME:
   Namer - A new cli application

USAGE:
   main [global options] command [command options] [arguments...]

VERSION:
   0.0.0

COMMANDS:
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version
```

詳細程式可以參考 [PR Day 16](https://github.com/MilesChou/namer/pull/1)

## 參考資料

* [dep][]

[dep]: https://github.com/golang/dep
[Dapper]: https://github.com/rancher/dapper
