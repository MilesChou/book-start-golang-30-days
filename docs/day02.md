# Environment

今天來建立開發環境，會分成安裝主程式與設定環境變數兩個部分。

## 安裝主程式

這裡的主程式是指 `go` 指令，它能處理編譯、直譯、建置、格式化程式碼、測試、下載依賴等多種工具的組合。

以下介紹常見的環境該如何安裝 go 。

### MacOS

使用 [Homebrew][] 可以簡單地安裝：

    $ brew install go
    $ go version
    go version go1.9.2 darwin/amd64

未來將會使用 MacOS 當作主要練習環境。

### Ubuntu Linux 16.04

使用預設安裝，但筆者試了一下， Ubuntu 預設版本是 Go 1.6 ：

> 使用 [Vagrant](https://www.vagrantup.com/) 建立 Ubuntu 虛擬環境測試

    $ vagrant init ubuntu/xenial64
    $ vagrant up
    $ vagrant ssh
    $ sudo apt-get install golang-go
    $ go version
    go version go1.6.2 linux/amd64

如果需要最新版，可以加入 golang 的 PPA ：

> 注意安裝套件與 go 指令的位置

    $ sudo add-apt-repository ppa:gophers/archive
    $ sudo apt update
    $ sudo apt-get install golang-1.9-go
    $ /usr/lib/go-1.9/bin/go version
    go version go1.9.2 linux/amd64

> 如安裝上遇到問題，也可參考 [wiki](https://github.com/golang/go/wiki/Ubuntu) 。

### Windows

可到[官方網站](https://golang.org/doc/install#windows)下載 MSI 檔安裝，接著重新打開命令提示字元後，就可以使用 go 指令了：

    C:\Users\User>go version
    go version go1.9.2 windows/386

## 環境變數設定

可下 `go env` 取得環境變數：

    $ go env
    GOARCH="amd64"
    GOBIN=""
    GOEXE=""
    GOHOSTARCH="amd64"
    GOHOSTOS="darwin"
    GOOS="darwin"
    GOPATH="/Users/miles.chou/go"
    GORACE=""
    GOROOT="/usr/local/Cellar/go/1.9.2/libexec"
    GOTOOLDIR="/usr/local/Cellar/go/1.9.2/libexec/pkg/tool/darwin_amd64"
    GCCGO="gccgo"
    CC="clang"
    GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/50/nzgqq0d96px715qlh926y2jcm2qxyy/T/go-build204908691=/tmp/go-build -gno-record-gcc-switches -fno-common"
    CXX="clang++"
    CGO_ENABLED="1"
    CGO_CFLAGS="-g -O2"
    CGO_CPPFLAGS=""
    CGO_CXXFLAGS="-g -O2"
    CGO_FFLAGS="-g -O2"
    CGO_LDFLAGS="-g -O2"
    PKG_CONFIG="pkg-config"

裡面有一個 `GOPATH` ，這是需要設定的環境變數。它代表著 go 程式的工作空間（workspace）， Windows 預設會設定在 `~\Go` ， Unix-like 則沒有預設，官方建議設定在 `~/go` 。

Workspace 裡，劃分成三個主要目錄：

* `src` - 原始碼
* `pkg` - go package
* `bin` - 編譯好的執行檔，有需要也可以加入 `PATH` 環境變數

接著可以使用 go 的第一個指令－－ `go get` ，它會把目標下載回來放在 src 裡，如：

    go get github.com/MilesChou/book-start-golang-30-days

這樣會把上面這個 repo ，使用 HTTPS 協定 clone 到硬碟裡。

當如果編譯需要第三方的原始碼時，即可使用 go get 下載，同時這也可以用來下載自己或是第三方的原始碼。把所有原始碼集中成一個大大的 workspace ，這就是 go 管理原始碼的概念。

## 今日回顧

* 安裝好主程式，設定好環境變數，明天就可以來 Hello World 了！

## 參考資料

* [Getting Started](https://golang.org/doc/install) | The Go Programming Language

[Homebrew]: https://brew.sh
