# The End

最後一天，再找個需求來做一下好了。

指令雖然完成了，但是下載下來如果使用者沒有 YAML 檔會無法使用。但理論上，程式應該提供這個檔案。

因此，今天來實作初始化 YAML 檔的指令。

## 分析

facade 定義一個新的函式，名稱就叫 `initial` 。

上次讀檔是用 `ioutil.ReadFile()` ，這次寫檔是 `ioutil.WriteFile()` 。

看樣子資訊足夠了。

## 開工

將初始化資訊放在 `provider/resource.go` 裡：

```go
const DefaultTemplate = `
lastNames:
- 李
- 王
- 張
- 劉
- 陳
- 楊
- 趙
- 黃

characterMale:
- 家
- 豪
- 志
- 明

characterFemale:
- 雅
- 婷
- 春
- 嬌
`
```

接著寫初始化的方法，記得要判斷檔案存不存在，不然好不容易改好內容後，又被覆寫會很不開心：

```go
func InitFileDefault(filename string) error {
	_, err := os.Stat(filename)
	if err != nil && os.IsNotExist(err) {
		return ioutil.WriteFile(filename, []byte(DefaultTemplate), 0644)
	}

	return nil
}
```

Command 的 Action 非常簡單：

```go
func(c *cli.Context) error {
    err := provider.InitFileDefault(c.GlobalString("provider"))

    return err
},
```

打完收工

---

好像有點太少，再做一點需求好了：前天實作 Command 的時候，忘了把單純產生名的方法加入。

設計簡單做就好。先加一個靜態變數 `GenerateFirstNameOnly` 在 facade 裡：
        
```go
var (
    // ...
    GenerateFirstNameOnly bool
)

func reset() {
    // ...
    GenerateFirstNameOnly = false
}
```

接著加一個 CLI 參數 `--first-name-only` 是布林值：

```go
cli.BoolFlag{
    Name:  "first-name-only",
    Usage: "只產生名",
},
```

CLI 布林值的取值非常簡單：

```go
facade.GenerateFirstNameOnly = c.Bool("first-name-only")
```

接著要調整 `provider/name.go` 產生名的函式。因為再加參數的話會太複雜，因此需要重構一下。把產生全名（`Name`）和只產生名（`NameFirstNameOnly`）的函式分開，然後產生全名的函式去使用產生名的函式即可：

```go
func (generator *Generator) Name(gender string, firstNameNum int) (name string, err error) {
	lastName, err := generator.NameFirstNameOnly(gender, firstNameNum)

	if err != nil {
		return
	}

	name = generator.LastName() + lastName

	return name, nil
}

func (generator *Generator) NameFirstNameOnly(gender string, firstNameNum int) (name string, err error) {
	switch gender {
	case "":
		name = generator.firstName(firstNameNum)
	case "male":
		name = generator.firstNameMale(firstNameNum)
	case "female":
		name = generator.firstNameFemale(firstNameNum)
	default:
		err = errors.New(fmt.Sprintf("gender '%s' is invalid", gender))
	}

	return
}
```

這樣 Facade 使用也會非常簡單，使用 if 判斷要使用哪個函式即可：

```go
if GenerateFirstNameOnly {
    name, err = generator.NameFirstNameOnly(GenerateGender, GenerateFirstNameNum)
} else  {
    name, err = generator.Name(GenerateGender, GenerateFirstNameNum)
}
```

再來 `ServeCommand` 實作也非常簡單，只要多設定一個 query string 就行了：

```go
firstNameOnly := g.DefaultQuery("firstNameOnly", "10")

if firstNameOnly == "1" {
    facade.GenerateFirstNameOnly = true
} else {
    facade.GenerateFirstNameOnly = false
}
```

## 展示

指令如下：

```
$ go run main.go generate --amount 3 --first-name-only
嬌志
婷婷
婷春
```

HTTP Server 如下：

```
$ curl "http://localhost:8080/generate?amount=3&firstNameOnly=1"
["明志","春志","婷嬌"]
```

原始碼可以參考 [PR Day 30](https://github.com/MilesChou/namer/pull/15)

## 最後的回顧

當初覺得 Go 非常難懂，最主要覺得難以理解的應該就是強型態的操作，與常數的表示方法。但在這 30 天裡有認真把它們翻過一次，然後多想幾個不一樣的範例試幾次後，現在漸漸有了解它的原理了。非常感謝良葛格詳細的筆記，讓我們能對 Go 的型態操作有較清楚的了解。

寫到現在，覺得 Go 確實蠻好上手的， 30 天就可以從只會 Hello World 到寫出一個簡單 CLI App ，雖然還缺少很多重要的觀念如測試、多執行緒設計等，但這就留給未來有機會再學了。

最後，很開心總算成功完成鐵人賽成就了！希望開發歷程的學習記錄，能幫助的到大家。對[鐵人賽文章](https://github.com/MilesChou/book-start-golang-30-days)或 [Namer](https://github.com/MilesChou/namer) 有任何建議或問題都歡迎發 issue ，謝謝大家！
