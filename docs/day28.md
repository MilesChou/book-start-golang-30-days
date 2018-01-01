# Add Command Parameters

我們在重構 [Name Provider][Day 26] 有提到，指令必須也要加參數，才有辦法傳給 Provider 產生對應的結果。

除此之外還有個需求：參考 [Faker](https://github.com/fzaninotto/Faker) ，我們還需要單純取得男性名字和女性名字的方法。

## 分析

先在 Name Provider 裡實作取得男性名字和女性名字後，再到指令的地方看該如何加參數。

另外，因為實作移到 Facade 了，所以可能要調整 Facade 的行為。

## 開工

使用最笨的方法：新增一堆公開函式來用，最原始的 `Name` 函式則新增了參數來決定背後要呼叫哪些函式：

```go
func (generator *Generator) Name(gender string, firstNameNum int) (name string, err error) {
	switch gender {
	case "":
		name = generator.LastName() + generator.firstName(firstNameNum)
	case "male":
		name = generator.LastName() + generator.firstNameMale(firstNameNum)
	case "female":
		name = generator.LastName() + generator.firstNameFemale(firstNameNum)
	default:
		err = errors.New(fmt.Sprintf("gender '%s' is invalid", gender))
	}

	return
}
```

`facade` 則加了兩個公開的靜態變數，來存放要帶入的值：

```go
var (
	GenerateFirstNameNum int
	GenerateGender string
)

func init() {
	reset()
}

func reset() {
	GenerateFirstNameNum = 2
	GenerateGender = ""
}
```

`Generate` 再去使用這些值帶入

```go
func Generate(path string, count int, process GenerateItemProcess) error {
	res, _ := provider.ParseFile(path)

	generator := provider.Create()
	generator.Resource = res

	for i := 0; i < count; i++ {
		name, err := generator.Name(GenerateGender, GenerateFirstNameNum)

		if err != nil {
			return err
		}

		process(name, i)
	}

	return nil
}
```

`ServeCommand` 也可以用 query string 帶入參數：

```go
func serve(c *cli.Context) error {
	server := gin.Default()
	server.GET(`/generate`, func(g *gin.Context) {
		num, err := strconv.Atoi(g.DefaultQuery("amount", "10"))

		if err != nil {
			g.JSON(400, err)
			return
		}

		firstNameNum, err := strconv.Atoi(g.DefaultQuery("firstNameNum", "0"))

		if err != nil {
			g.JSON(400, err)
			return
		}

		facade.GenerateFirstNameNum = firstNameNum
		facade.GenerateGender = g.DefaultQuery("gender", "")

		s := make([]string, num)
		facade.Generate(c.GlobalString("provider"), num, func(item string, index int) {
			s[index] = item
		})

		g.JSON(200, s)
	})

	server.Run()

	return nil
}
```

## 展示

現在應該可以自由使用參數調整輸出的結果了，如：

```
$ go run main.go generate --amount 1 --first-name-num 2 --gender male
楊明豪
$ go run main.go generate --amount 2 --first-name-num 2 --gender female
李雅嬌
劉嬌婷
```

HTTP server 也可以用 query string 調整，如：

```
$ curl "http://localhost:8080/generate?amount=1&firstNameNum=2&gender=male"
["陳明志"]
$ curl "http://localhost:8080/generate?amount=2&firstNameNum=2&gender=female"
["李嬌雅","劉婷春"]
```

程式碼可以參考 [PR Day 28](https://github.com/MilesChou/namer/pull/14)

## 參考資料

* [Faker](https://github.com/fzaninotto/Faker)

[Day 26]: /docs/day26.md
