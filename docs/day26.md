# Refactoring Name Provider

前面 25 天，我們已經成功寫出了一個 CLI App 以及 Web App ，包括交付與佈署都有實作，這次鐵人賽主題的基本要求已經算達標了。

剩下五天的目標將會是改善這個程式，無論是功能上的改進或是[重構][看到 code 寫成這樣我也是醉了，不如試試重構？]。

首先有一個問題非常明顯：台灣人的名通常都是兩個字，我們必須保留一點彈性在字數調整上。

## 分析

這個問題一直拖著沒改，其實是因為要先重構原本的程式碼，才會好動工。

原本的 `provider/name.go` 的程式碼片段如下：

```go
func (generator *Generator) LastName() string {
	length := len(generator.Resource.LastNames)
	randomIndex := generator.rand.Intn(length)

	return generator.Resource.LastNames[randomIndex]
}

func (generator *Generator) FirstName() string {
	merge := append(generator.Resource.CharacterMale, generator.Resource.CharacterFemale...)
	length := len(merge)
	randomIndex := generator.rand.Intn(length)

	return merge[randomIndex]
}
```

我們可以發現， `LastName` 與 `FirstName` 有一個模式是一樣的：在某個 collection 裡面隨便取一筆資料。

因此第一步可以先把這個行為抽出，寫成另一個函式 `pickCharacter` ：

```go
func (generator *Generator) LastName() string {
	return generator.pickCharacter(generator.Resource.LastNames)
}

func (generator *Generator) FirstName() string {
	merge := append(generator.Resource.CharacterMale, generator.Resource.CharacterFemale...)

	return generator.pickCharacter(merge)
}

func (generator *Generator) pickCharacter(collection []string) string {
	length := len(collection)
	randomIndex := generator.rand.Intn(length)

	return collection[randomIndex]
}
```

再來，台灣人的名大部分是兩個字，但姓和單名會是一個字。或許 `pickCharacter` 多一個參數來決定要取幾個字會是個好選擇。先把參數加進去試看看：

```go
func (generator *Generator) LastName() string {
	return generator.pickCharacter(generator.Resource.LastNames, 1)
}

func (generator *Generator) FirstName() string {
	merge := append(generator.Resource.CharacterMale, generator.Resource.CharacterFemale...)

	return generator.pickCharacter(merge, 2)
}

func (generator *Generator) pickCharacter(collection []string, count int) (chars string) {
	length := len(collection)

	for i := 0; i < count; i++ {
		randomIndex := generator.rand.Intn(length)
		chars = chars + collection[randomIndex]
	}

	return chars
}
```

接著要開放這個數字讓外界可以呼叫，才會有實際的價值，這裡的做法是先把原本的 `FirstName` 改名為 `firstName` 並加入 `count` 參數，接著開出三個接口 `FirstName` 、 `FirstNameSingle` 、 `FirstNameDouble` ：

```go
func (generator *Generator) FirstName() string {
	return generator.firstName(generator.rand.Intn(2) + 1)
}

func (generator *Generator) FirstNameSingle() string {
	return generator.firstName(1)
}

func (generator *Generator) FirstNameDouble() string {
	return generator.firstName(2)
}

func (generator *Generator) firstName(count int) string {
	merge := append(generator.Resource.CharacterMale, generator.Resource.CharacterFemale...)

	return generator.pickCharacter(merge, count)
}
```

原本的 `Name` 函式行為改成會亂數取單名與複名。如果要指定單名或複名的話，我們必須再加開兩個函式 `NameSingle` 與 `NameDouble` ：

```go
func (generator *Generator) NameSingle() string {
	return generator.LastName() + generator.FirstNameSingle()
}

func (generator *Generator) NameDouble() string {
	return generator.LastName() + generator.FirstNameDouble()
}
```

指令也需要重構，明天再針對指令做調整。

## 展示

```
$ go run main.go generate
Generate 10
楊志雅
劉明志
張家豪
趙春
趙志婷
王春春
劉志
趙嬌家
李志
王婷
```

程式碼可以參考 [PR Day 26](https://github.com/MilesChou/namer/pull/12)

## 參考資料

* [看到 code 寫成這樣我也是醉了，不如試試重構？][]

[看到 code 寫成這樣我也是醉了，不如試試重構？]: https://github.com/MilesChou/book-refactoring-30-days
