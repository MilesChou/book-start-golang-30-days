# Commands and Flags

在開始正式寫假文產生器前，我們先來看看哪些子命令和參數是需要定義的。

## 分析

找了一下套件說明，看起來只要把這個值代入 Command 結構的 slice 即可有子命令：

```go
app.Commands = []cli.Command{}
```

另外還會需要參數，比方說一次想要產生的數量有多少， `app` 和 `Command` 都有一個值域叫 Flags ，只要給它 Flag 結構的 slice 即可：

```go
app.Flags = []cli.Flag{
    cli.StringFlag{
        Name:  "num",
        Value: "10",
        Usage: "產生數量",
    },
}
```

取 Flags 的方法如下：

```go
func(c *cli.Context) error {
    fmt.Println(c.String("num"))

    return nil
}
```

## 開工

先定義兩個子命令 `generate` 與 `status` ，而 `generate` 定義一個 flags 是 `num` ，另外把它抽出另一個函式初始化：

```go
package main

import (
	"fmt"
	"os"
	"github.com/urfave/cli"
)

func main() {
	app := cli.NewApp()
	app.Name = "Namer"
	app.Commands = commands()

	app.Run(os.Args)
}

func commands() []cli.Command {
	return []cli.Command{
		{
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
				fmt.Println("Hello Generate")
				fmt.Println("Generate " + c.String("num"))

				return nil
			},
		},
		{
			Name:  "status",
			Usage: "狀態",
			Action: func(c *cli.Context) error {
				fmt.Println("Hello Status")

				return nil
			},
		},
	}
}
```

## 展示

執行：

```bash
$ go run main.go generate
Hello Generate
Generate 10

$ go run main.go generate --num 1000
Hello Generate
Generate 1000
```

Flags 和 Args 好像只能吃字串，不過這問題並不大，而且也蠻正常的，之後再來解吧。

詳細程式可以參考 [PR Day 17](https://github.com/MilesChou/namer/pull/2)