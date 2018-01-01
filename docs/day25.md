# Docker

做完交付後，下一個目標就是要做佈署了！不過佈署做簡單一點，在 Docker 上能跑就行了！

最後期望的結果是，只要機器有 Docker Daemon ，程式就能正常在機器上跑！

## 分析

理論上， Go 執行檔的依賴都在執行檔上，或許放到 [alpine](https://hub.docker.com/_/alpine/) 上是可行的

來試看看吧！

## 開工

先編譯過檔案：

```
$ go build
```

然後進到 `alpine` 裡：

```
$ docker run --rm -it -v `pwd`:/source alpine sh
# --- Docker Container 裡 ---
$ cd /source
$ ./namer
./namer: line 1: �����: not found
./namer: line 2: can't open Ԓ��: no such file
./namer: line 2: H__PAGEZEROx__TEXTЖЖ__text__TEXT�S__rodata__TEXT: not found
./namer: line 14: syntax error: unexpected "("
```

失敗了，可能是缺少了什麼東西，我們換最肥的 `ubuntu` 試試：

```
$ docker run --rm -it -v `pwd`:/source ubuntu bash
# --- Docker Container 裡 ---
$ cd /source
$ ./namer 
bash: ./namer: cannot execute binary file: Exec format error
```

咦，這不科學啊！不是說好了依賴都放在裡面了嗎？後來才想到，原來是因為筆者編譯環境是 Mac ，才會造成這樣的結果。

雖然可以把昨天丟到 GitHub Release 的結果下載回來，在放到 Docker 裡面，不過這也太麻煩了。原始碼都在手上了，應該產生執行檔會是件簡單的事呀！

Go 有支援 cross compile ，不過目前只讓它在 Travis 上運行，我們來用 [Dapper][] 實現目的。

### 解決 Dapper 的問題

首先，因為 Mac 前一陣子更新，導致 [Dapper 官方提供的可執行檔不能用了](https://github.com/rancher/dapper/issues/47)。

一種方法是使用 [Dapper Image](https://hub.docker.com/r/rancher/dapper/) 建立可以用 Dapper 的環境。這個方法比較麻煩，更重要的是，我們都已經學 Go 了，直接抓下來編譯一下就可以了呀！

所以我們來編譯它吧，首先把 Dapper Clone 下來：

```
$ go get github.com/rancher/dapper
``` 

然後切換到目錄下

```
$ cd ${GOPATH}/src/github.com/rancher/dapper
```

下安裝指令

```
$ go install
```

> 或是直接下安裝指令 `go install github.com/rancher/dapper` 也可以，上面就是它實際會做的事。

接著應該就會發現 `${GOPATH}/bin` 有 dapper 這個檔案了：

```
$ ls ${GOPATH}/bin
dapper
```

只要把環境變數 `PATH` 指定好，就可以用 dapper 了：

```
PATH=$PATH:$GOPATH/bin
```

> 同樣的概念， Namer 裡使用 `go install` 也會多一個指令是 `namer` 唷。

### 撰寫 Dapper

Dapper 的用法可以參考[官網][Dapper]或是[教學文章][自己來的好選擇 －－ Dapper]，這邊就不贅述了。

寫完內容如下：

```dockerfile
FROM golang:1.9-alpine

ENV DAPPER_SOURCE /go/src/github.com/MilesChou/namer
ENV DAPPER_OUTPUT ./bin
RUN mkdir -p ${DAPPER_SOURCE}
WORKDIR ${DAPPER_SOURCE}

CMD ["go", "build", "-o", "./bin/namer", "main.go"]
```

這次 build 出來的 namer 就能正常在 Alpine 下執行了。

### 撰寫 Dockerfile

因為只有 `namer` 執行檔就行， Dockerfile 相對非常簡單，如下：

```dockerfile
FROM alpine:3.7

RUN mkdir -p /source
WORKDIR /source
COPY ./bin/namer ./
COPY names.yml ./

EXPOSE 8080

ENTRYPOINT ["./namer"]
CMD ["help"]
```

## 展示

Docker Build 與 Docker Run 的範例：

```
$ docker build -t namer .
$ docker run -it --rm namer --version
Namer version 0.0.0
```

程式碼可以參考 [PR Day 25](https://github.com/MilesChou/namer/pull/11)

## 問題

在建置的過程發現， `namer` 執行檔一個就要 14MB 之多，有點可怕。

暫時還不知道該如何解決，目前先讓它可以用就好。

## 參考資料

* [Dapper][]
* [自己來的好選擇 －－ Dapper][] | CI 從入門到入坑

[Dapper]: https://github.com/rancher/dapper
[自己來的好選擇 －－ Dapper]: https://ithelp.ithome.com.tw/articles/10187177
