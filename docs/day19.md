# File

文字清單如果都寫死在程式裡的話，擴充性就太差了，預期它應該要可以從檔案抓出文字清單。

## 分析

基本的檔案操作應該不大會有問題，要思考的會是，該要用什麼樣的格式來存放文字清單？

另外程式應該要從哪載入文字清單？固定位置？或是 Flags 參數帶給程式？

格式的部分，會以好閱讀與修改為主，因此會選擇 [YAML](http://www.yaml.org/) ，套件使用 [`go-yaml`](https://github.com/go-yaml/yaml) ，載入路徑會使用 Flags 參數帶入。

`go-yaml` 試了一下，它支援輸出 Struct 或是 Map 。 Struct 的參數必須要定義公開，而且要跟 YAML 格式相符，不然會存放失敗， Map 則是進來什麼都吃，沒有這些限制。目前情境還很單純，使用 Struct 是個可行的選擇。

但它只吃字串，所以必須要寫一段讀檔程式，今天就來試試 Flags 參數加讀檔串接吧。

## 開工

因為 command 要做的事，目前是硬塞到 `main.go` 裡，這樣會違反[單一職責原則][Refactoring Day 7]。在開始前先重構，把職責分離清楚，不然後面應該會更難搞。

做法很簡單：開一個 `command` 的目錄，新增兩個 `generate.go` 與 `status.go` ，把 Command 原本要給的值，換到這兩個檔案裡面定義即可。

`command/generate.go` 的內容如下：

```go
var (
	GenerateCommand = cli.Command{
		Name:  "generate",
		Usage: "產生假名",
		Flags: []cli.Flag{
			cli.StringFlag{
				Name:  "num",
				Value: "10",
				Usage: "產生數量",
			},
		},
		Action: func(c *cli.Context) error {
			return generate(c)
		},
	}
)

func generate(c *cli.Context) error {
	num, err := strconv.Atoi(c.String("num"))

	if err != nil {
		return err
	}

	fmt.Println("Generate " + strconv.Itoa(num))

	generator := provider.Create()

	for i := 0; i < num; i++ {
		fmt.Println(generator.Name())
	}

	return nil
}
```

`command/status.go` 的內容目前只是樣版，如下：

```go
package command

import (
	"fmt"
	"github.com/urfave/cli"
)

var (
	StatusCommand = cli.Command{
		Name:  "status",
		Usage: "狀態",
		Action: func(c *cli.Context) error {
			fmt.Println("Hello Status")

			return nil
		},
	}
)
```

這樣 `main.go` 就會變得非常簡單：

```go
package main

import (
	"os"
	"github.com/MilesChou/namer/command"
	"github.com/urfave/cli"
)

func main() {
	app := cli.NewApp()
	app.Name = "Namer"
	app.Commands = []cli.Command{
		command.GenerateCommand,
		command.StatusCommand,
	}

	app.Run(os.Args)
}
```

詳細重構程式可以參考 [PR Day 19 前重構](https://github.com/MilesChou/namer/pull/4)

---

下面開始實作參數與讀檔。撞了很多牆後，參考[別人的做法](https://github.com/rancher/dapper/blob/master/main.go#L104-L108)，發現應該是需要先 `os.Chdir()` 後，再 `os.Stat()` 就能找得到了，來試看看：

先建立 `names.yml` 檔案，然後在 `command/status.go` 的 Action 輸入下面的程式

```go
os.Chdir(".")
os.Stat("names.yml")
fmt.Println(os.Stat("names.yml"))
```

輸出：

```
&{names.yml 5 420 {579465000 63650306753 0x121e060} {16777220 33188 1 7295335 673970142 1079850989 0 [0 0 0 0] {1514709976 915571588} {1514709953 579465000} {1514709953 579483767} {1514709952 195533478} 5 8 4194304 0 0 0 [0 0]}} <nil>
```

看來是可行的，接著使用 [`io/ioutil`](https://golang.org/pkg/io/ioutil/) 來取得 byte 資料：

```go
os.Chdir(".")
ra, _ :=  ioutil.ReadFile("names.yml")

fmt.Println(`------ File Content Start ------`)
fmt.Printf("%s", ra)
fmt.Println(`------- File Content End -------`)
```

輸出：

```go
------ File Content Start ------
Hello File!
------- File Content End -------
```

下一步把傳入的檔案參數化，使用 `string` 格式：

```go
Flags: []cli.Flag{
    cli.StringFlag{
        Name:  "provider",
        Value: "names.yml",
        Usage: "名字倉庫",
    },
}
```

最後 Action 的函式會長這樣：

```go
Action: func(c *cli.Context) error {
    os.Chdir(".")

    r, _ := ioutil.ReadFile(c.String("provider"))

    fmt.Println(`------ File Content Start ------`)
    fmt.Printf("%s", r)
    fmt.Println(`------- File Content End -------`)

    return nil
}
```

大功告成！

詳細程式可以參考 [PR Day 19](https://github.com/MilesChou/namer/pull/5)

## 問題

讀檔應該會是全域的設定功能，因此下次要開始前，必須要先重構這部分的程式。

## 參考資料

* [SOLID 之 單一職責原則（Single responsibility principle）][Refactoring Day 7] | 看到 code 寫成這樣我也是醉了，不如試試重構？

[Refactoring Day 7]: https://github.com/MilesChou/book-refactoring-30-days/blob/master/docs/day07.md
 