# Delivery

截至目前為止，應用程式該有基本功能都已經完備了，再來就是最後一哩路了－－交付。

## 分析

交付前必須要經過建置（[Build][]）的過程，不過畢竟只是 side project ，所以測試暫時先跳過吧！但有一個很重要的任務不能不做－－**編譯**。

Go 的編譯還算簡單，直接下指令 build 就行了

```
$ go build
```

接著就會出現與專案名稱同名的可執行檔，如 `namer` ：

```
$ ls namer
namer
```

交付的目標，應該直接放在 [GitHub Release](https://github.com/blog/1547-release-your-software) 即可。

Travis CI 的串接方法可以參考[官方網站](https://docs.travis-ci.com/user/deployment/releases/)。

Travis CI 上做 cross compile 可以參考 [gimme](https://github.com/travis-ci/gimme#travisyml) 。

## 開工

首先要先安裝 [travis 指令](https://github.com/travis-ci/travis.rb#installation) ，安裝好應該就能看得到版本號了：

```
$ travis -v
1.8.8
```

接著切換專案目錄，執行下面的指令：

```
$ travis setup releases
Detected repository as MilesChou/namer, is this correct? |yes| 
Username:
Password for jangconan@gmail.com:
File to Upload: namer
Deploy only from MilesChou/namer? |yes| 
Deploy from iron branch? |yes| no
Encrypt API key? |yes| 
```

它會問一些問題，其中記得 `Encrypt API key` 一定要選 yes ，因為這個 key 是機敏資訊。 

接著調整 `.travis.yml` 描述檔：

```yaml
branches:
  only:
  - master
  - /^\d+\.\d+\.\d+$/

env:
- GIMME_OS=linux GIMME_ARCH=amd64
- GIMME_OS=darwin GIMME_ARCH=amd64
- GIMME_OS=windows GIMME_ARCH=amd64

before_deploy:
- go build

deploy:
  provider: releases
  api_key:
    secure: ENCRYPT_API_KEY
  file: namer
  skip_cleanup: true
  on:
    tags: true
```

一切就緒，當打 tag 的時候，就會觸發 deploy 行為，然後把建置出來的執行檔都交付到 GitHub Release 。

程式碼可以參考 [PR Day 24](https://github.com/MilesChou/namer/pull/10) 。

## 參考資料

* [CI 起步走][Build] | CI 從入門到入坑

[Build]: https://github.com/MilesChou/book-intro-of-ci/blob/release/docs/day06.md
