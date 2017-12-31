# Send HTTP Request

中國字非常深奧，有些字的含意，有時候並不是那麼清楚。還好網路上都能查得到這些資訊。

今天要來做如何把查出來的網頁資訊抓下來，也就是常見的 HTTP 協定功能。

## 分析

首先要先有目標網站，以網址簡單來說的話，[萌典][]應該是好選擇。

套件本來想找 Curl ，不過想想還是找簡單的就好： [GoReq](https://github.com/franela/goreq)

看說明應該非常簡單：

```go
res, err := goreq.Request{ Uri: "http://www.google.com" }.Do()
```

可以來實作了！

## 開工

先在 `provider` 裡實作查字典功能：

```go
package provider

import "github.com/franela/goreq"

const (
	dictionary = `https://www.moedict.tw/`
)

func Query(word string) (str string, err error){
	req := goreq.Request{
		Uri: dictionary + string(word),
	}

	res, err := req.Do()
	if err != nil {
		return
	}

	return res.Body.ToString()
}
```

先把查到的網頁回傳出去，明天再來處理正則。

而原本 `GenerateCommand` 有點亂，先不要在上面加功能，另外開一個 `QueryCommand` 來呼叫這個 `Query` 方法：

```go
package command

import (
	"fmt"
	"github.com/MilesChou/namer/provider"
	"github.com/urfave/cli"
)

var (
	QueryCommand = cli.Command{
		Name:  "query",
		Usage: "查詢字典",
		Action: func(c *cli.Context) error {
			return query(c)
		},
	}
)

func query(c *cli.Context) error {
	str := c.Args().First()

	query, err := provider.Query(str)

	if err != nil {
		fmt.Println(err)
		return err
	}

	fmt.Println(query)

	return nil
}
``` 

`main.go` 的程式碼也貼一下騙版面：

```go
app.Commands = []cli.Command{
    command.GenerateCommand,
    command.StatusCommand,
    command.QueryCommand,
}
```

因為依賴有調整，需要下指令更新 lock 檔：

```
$ dep ensure
```

> [YAML][Day 20] 那次也有做，只是忘了提這件事

## 展示

今天做的東西還蠻單純的，展示如下：

```
$ go run main.go query 字
<!DOCTYPE html>
...
</html>
```

程式碼可以參考 [PR Day 21](https://github.com/MilesChou/namer/pull/7)

## 問題

發現裡面都是 head 的區塊，沒有 body ，看來要爬這個網站會有點麻煩。

## 參考資料

* [萌典][]

[萌典]: https://www.moedict.tw
[Day 20]: /docs/day20.md
