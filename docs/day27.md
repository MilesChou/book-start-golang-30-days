# Refactoring Command

指令套件 [`github.com/urfave/cli`](https://github.com/urfave/cli) 算容易上手的。雖然好用，但似乎其他套件也不錯，如 [Cobra](https://github.com/spf13/cobra) 等。

但目前 Command 實際處理任務的程式都是直接綁死在 `cli.Context` 上，這樣就會違反[依賴反轉原則][SOLID 之 依賴反轉原則（Dependency inversion principle）]－－應該依賴更抽象的參數，如 [*Predeclared Type*][Day 6] 或 interface 等。

會發現這個問題是因為在實作 [HTTP Server][Day 23] 時，行為都沒有變，只有輸出改變而已，但卻必須要為 `/generate` 的回傳重新客製化寫法，而且這段程式碼與 `command/generate.go` 裡面的 `generate` 函式太像了，這是一個明顯的[壞味道][開發者能察覺的壞味道（Bad Smell）]，必須要重新調整設計才行。

## 分析

首先先調整 `command/generate.go` 的 `generate` 函式。它的任務是產生一堆假資料，所以我們應該可以先把數量這個參數先抽離出來：

```go
func generate(count int, c *cli.Context) error {
	res, _ := provider.ParseFile(c.GlobalString("provider"))

	generator := provider.Create()
	generator.Resource = res

	for i := 0; i < count; i++ {
		fmt.Println(generator.Name())
	}

	return nil
}
```

接著取得 `res` 是 provider 的任務，因此它不適合離開這個範圍，但取得檔案名稱的 `c.GlobalString("provider")` ，是可以抽離的：

```go
func generate(path string, count int) error {
	res, _ := provider.ParseFile(path)

	generator := provider.Create()
	generator.Resource = res

	for i := 0; i < count; i++ {
		fmt.Println(generator.Name())
	}

	return nil
}
```

最後就是 `fmt.Println(generator.Name())` 這是 cli 專屬的行為，因此把它抽出變成 [Closure][Day 12] ，同時把這個方法也公開：

```go
type GenerateItemProcess func(item string)

func Generate(path string, count int, process GenerateItemProcess) error {
	res, _ := provider.ParseFile(path)

	generator := provider.Create()
	generator.Resource = res

	for i := 0; i < count; i++ {
		process(generator.Name())
	}

	return nil
}
```

到目前為止，只要把上面的程式碼找個地方放即可。因為這有點類似 *Facade Pattern* ，因此決定放在 `facade/facade.go` 裡。原本 CLI 呼叫的地方改成這樣：

```go
func(c *cli.Context) error {
    num, err := strconv.Atoi(c.String("num"))

    if err != nil {
        return err
    }

    fmt.Println("Generate " + strconv.Itoa(num))

    return facade.Generate(c.GlobalString("provider"), num, func(item string) {
        fmt.Println(item)
    })
}
```

同時把不必要的程式碼刪除， import 的項目就不需要 `provider` 了，改成依賴 `facade` 。 HTTP Server 實作的部分也可以如法炮製：

```go
func serve(c *cli.Context) error {
	num := 10

	server := gin.Default()
	server.GET(`/generate`, func(g *gin.Context) {
		var s []string

		facade.Generate(c.GlobalString("provider"), num, func(item string) {
			s = append(s, item)
		})

		g.JSON(200, s)
	})

	server.Run()

	return nil
}
```

同個重構方法也可以用在 `QueryCommand` ，後面就不贅述了。

> 另外兩個 Command ： `ServeCommand` 目前不知道該怎麼拆出來好（因為裡面也有用到 Facade ）； `StatusCommand` 則是因為太簡單，所以沒拆出來的必要。

### 效能改善

以上已經把 `Generate` 的任務解耦合了，但 HTTP Server 的效能上是有問題的，當 num 到了一個極大的數如 10,000,000 ，執行會需要花 7 秒：

```
[GIN] 2018/01/01 - 23:37:19 | 200 |  7.610408045s |       127.0.0.1 |  GET     /generate
```

主要是因為 `append` 了一千萬次，我們把 `Generate` 程式改一下：

```go
type GenerateItemProcess func(item string, index int)

func Generate(path string, count int, process GenerateItemProcess) error {
	res, _ := provider.ParseFile(path)

	generator := provider.Create()
	generator.Resource = res

	for i := 0; i < count; i++ {
		process(generator.Name(), i)
	}

	return nil
}
```

讓 process 可以順便把目前處理到第幾個，也傳入 Closure ， HTTP Server 也能改寫成這樣：

```go
s := make([]string, num)

facade.Generate(c.GlobalString("provider"), num, func(item string, index int) {
    s[index] = item
})
```

這樣算是用空間換時間，時間結果約為原本的 70% ：

```
[GIN] 2018/01/01 - 23:41:34 | 200 |  4.963947486s |       127.0.0.1 |  GET     /generate
```

程式碼修改可以參考 [PR Day 27](https://github.com/MilesChou/namer/pull/13)

## 問題

Interface 沒認真去了解，還真不知道該怎麼做好。

明天把 Command 的參數加完後，後天就來研究 Interface ！

## 參考資料

* [SOLID 之 依賴反轉原則（Dependency inversion principle）][] | 看到 code 寫成這樣我也是醉了，不如試試重構？
* [開發者能察覺的壞味道（Bad Smell）][] | 看到 code 寫成這樣我也是醉了，不如試試重構？
* [Facade 模式](https://openhome.cc/Gossip/DesignPattern/FacadePattern.htm) | 良葛格學習筆記

[開發者能察覺的壞味道（Bad Smell）]: https://github.com/MilesChou/book-refactoring-30-days/blob/master/docs/day04.md
[SOLID 之 依賴反轉原則（Dependency inversion principle）]: https://github.com/MilesChou/book-refactoring-30-days/blob/master/docs/day11.md
[Day 6]: /docs/day06.md
[Day 12]: /docs/day12.md
[Day 23]: /docs/day23.md
