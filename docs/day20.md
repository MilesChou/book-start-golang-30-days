# YAML

昨天已經成功把檔案載入變成 `[]byte` 型態，今天要來讀 YAML 檔了。

## 分析

昨天有提到會使用 `go-yaml` 解析 YAML 資料，資料格式參考 [Faker](https://github.com/fzaninotto/Faker/blob/v1.7.1/src/Faker/Provider/zh_TW/Person.php) ，大概會長像下面這樣：

```yaml
lastNames:
- 李
- 王
- 張
- 劉
- 陳
- 楊
- 趙
- 黃
- ...

characterMale:
- 家
- 豪
- 志
- 明
- ...

characterFemale:
- 雅
- 婷
- 春
- 嬌
- ...
```

`go-yaml` 的用法大概如下：

```go
s := NamesProvider{}
yaml.Unmarshal(bytes, s)

fmt.Println(s) // 印出解析完的 struct
```

這樣應該就可以開始實作了。

## 開工

開始前，一樣要先重構：把切目錄和讀檔的任務放到最外層（`main.go`）。

不過因為不大清楚 CLI 套件怎麼做全域的任務，所以不如就把解析 YAML 的任務先放在 `resource` 裡：

```go
type NamesResource struct {
	LastNames       []string `yaml:"lastNames"`
	CharacterMale   []string `yaml:"characterMale"`
	CharacterFemale []string `yaml:"characterFemale"`
}

func ParseFile(file string) (res NamesResource, err error) {
	r, err := ioutil.ReadFile(file)
	if err != nil {
		return res, err
	}

	if err := yaml.Unmarshal(r, &res); err != nil {
		return res, err
	}

	return res, nil
}
```

其中需注意的是， go-yaml 預設會先把 field 轉全小寫，再去 YAML 的資料裡面找。而 LastNames field 預設會找 `lastnames` 的欄位，然後就找不到，這時會需要用 `yaml:"lastNames"` 指定要找的欄位。

其他兩個 Command 開頭先執行吧！之後有空再來想想該怎麼重構：

```go
t, _ := provider.ParseFile(c.GlobalString("provider"))

fmt.Println(t)
```

---

今天這樣太簡單了。資料都有了，不如就把亂數取名實作出來吧！

回憶一下之前取名的方法（順便騙版面）：

```go
func (generator *Generator) Name() string {
	length := len(names)

	return names[generator.rand.Intn(length)]
}
```

這裡是用 `generator.rand` 取得亂數，再去口袋名單裡面取。但現在口袋名單變成了 `NameResource` ，最簡單的做法就是在 Generator 加一個 Resource field ：

```go
type Generator struct {
	rand *rand.Rand
	Resource NamesResource
}
```

然後直接在 Command Action 指定：

```go
res, _ := provider.ParseFile(c.GlobalString("provider"))

generator := provider.Create()
generator.Resource = res
```

這樣裡面就會有資料可以用了。接著參考 [Faker](https://github.com/fzaninotto/Faker/blob/v1.7.1/src/Faker/Provider/Person.php) 的做法，它會把姓跟名分開，實作如下：

```go
func (generator *Generator) Name() string {
	return generator.LastName() + generator.FirstName()
}

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

## 展示

執行結果如下：

```
$ go run main.go generate
Generate 10
楊雅
李婷
黃明
劉雅
陳明
楊豪
黃春
劉家
王嬌
趙家
```

詳細程式可以參考 [PR Day 20](https://github.com/MilesChou/namer/pull/6)

## 參考資料

* [Faker](https://github.com/fzaninotto/Faker)
* [Faker（1）－－假文產生器](https://github.com/MilesChou/book-decompose-wheels/blob/master/docs/day06.md) | 輪子們，聽口令，大部分解開始！
