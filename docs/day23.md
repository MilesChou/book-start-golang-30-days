# HTTP Server

因為參加的是 Modern Web 主題，不管怎樣，還是跟 Web 掛勾一下好了。

今天的主題是如何起一個 Web Server 。

## 分析

Go 本身即有內帶一些可用的函式庫，廣大的 [GitHub][Awesome Go] 上也有非常多套件可以參考。今天會使用 [Gin](https://github.com/gin-gonic/gin) 。

Gin 使用很簡單，官方範例如下：

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // listen and serve on 0.0.0.0:8080
}
```

啟動 Server 的方法就學 Laravel [Artisan][] 好了：

```
$ php artisan serve
```

API 設計直接以 command 的功能命名，先來做 generate 就好。

預計的效果如下：

```
GET /generate
[
    "張春",
    "李明",
    "劉家",
    "張志",
    "劉雅",
    "楊雅",
    "陳豪",
    "楊嬌",
    "劉明",
    "劉春"
]
```

## 開工

首先先把範例程式碼加入 Command ：

```go
package command

import (
	"github.com/urfave/cli"
	"github.com/gin-gonic/gin"
)

var (
	ServeCommand = cli.Command{
		Name:  "serve",
		Usage: "啟動伺服器",
		Action: func(c *cli.Context) error {
			return serve(c)
		},
	}
)

func serve(c *cli.Context) error {
	server := gin.Default()
	server.GET(`/generate`, func(c *gin.Context) {
		c.JSON(200, gin.H{
			"result": "ok",
		})
	})

	server.Run()

	return nil
}
```

這樣就能正常的打 `/generate` API 了

```
$ go run main.go serve
$ curl 127.0.0.1:8080/generate
{"result":"ok"}
```

再來把 `GenerateCommand` 的實作搬來 `serve` 函式：

```go
func serve(c *cli.Context) error {
	res, _ := provider.ParseFile(c.GlobalString("provider"))
	num := 10

	server := gin.Default()
	server.GET(`/generate`, func(c *gin.Context) {
		generator := provider.Create()
		generator.Resource = res

		s := []string{}
		for i := 0; i < num; i++ {
			s = append(s, generator.Name())
		}

		provider.Create()
		c.JSON(200, s)
	})

	server.Run()

	return nil
}
```

> `num` 先硬寫，後面再調整。

大功告成！

## 問題

上面的程式碼可以注意到： `c.JSON` 帶的第二個參數似乎是可以任意值的，這應該是 `interface` 的特色。

YAML 在實作的時候也一直看到 `interface` ，所以看來有機會還是得好好研究一下它。

## 展示

```
$ go run main.go serve
$ curl 127.0.0.1:8080/generate
["李雅","王家","李婷","黃家","張嬌","李志","陳豪","楊婷","黃婷","楊婷"]
```

程式碼可以參考 [PR Day 23](https://github.com/MilesChou/namer/pull/9)

## 參考資料

* [Awesome Go][]
* [Interface types](https://golang.org/ref/spec#Interface_types) | The Go Programming Language Specification

[Awesome Go]: https://awesome-go.com/
[Artisan]: https://laravel.com/docs/5.5/artisan
