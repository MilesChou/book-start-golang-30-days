# Parse JSON

昨天把網頁載好，不過裡面的資料似乎很難處理。後來有找到另一個 API ：

```
https://www.moedict.tw/a/字.json
```

它會回傳 JSON 格式的字串，解析簡單很多。而 [YAML][Day 20] 套件也能解析 JSON ，直接上吧！

## 開工

首先先把 URL 改掉：

```go
const (
	dictionary = `https://www.moedict.tw/a/%s.json`
)
```

呼叫改使用 `fmt.Sprintf()` ：

```go
req := goreq.Request{
    Uri: fmt.Sprintf(dictionary, string(word)),
}
```

這樣執行應該回傳就會變成 JSON 格式了。

下一步是要把 JSON 轉成 go 的資料型態，這次的結構就很鬆散了，改用 Map 做會比較簡單：

```go
type MoeDict struct {
	Heteronyms []Heteronym
}

type Heteronym struct {
	Definitions []string
}

func convertJsonToStruct(json string) (dict MoeDict, err error) {
	m := make(map[interface{}]interface{})
	yaml.Unmarshal([]byte(json), &m)

	heteronyms := convertHeteronyms(m["h"])

	return MoeDict{heteronyms}, err
}

func convertSliceMap(sliceMap []interface{}) (sm []map[interface{}]interface{}) {
	for _, m := range sliceMap {
		sm = append(sm, m.(map[interface{}]interface{}))
	}

	return sm
}

func convertHeteronyms(in interface{}) (out []Heteronym) {
	heteronyms := convertSliceMap(in.([]interface{}))

	for _, heteronym := range heteronyms {
		definitions := convertDefinitions(heteronym["d"])

		out = append(out, Heteronym{
			definitions,
		})
	}

	return
}

func convertDefinitions(in interface{}) (out []string) {
	definitions := convertSliceMap(in.([]interface{}))

	for _, definition := range definitions {
		def := convertDef(definition["f"].(string))

		out = append(out, def)
	}

	return
}

func convertDef(in string) string {
	def := in

	def = strings.Replace(def, "~", "", -1)
	def = strings.Replace(def, "`", "", -1)

	return def
}
```

因為原始資料的設計是多讀音配多種解釋，會是樹狀結構加一堆陣列，所以必須要很多 `for` 來處理。

而 QueryCommand 改寫成這樣：

```go
dict, err := provider.Query(str)

for _, heteronym := range dict.Heteronyms {
    for _, definition := range heteronym.Definitions {
        fmt.Println(definition)
    }
}
```

## 展示

```
$ go run main.go query 萌
草木初生的芽。
事物發生的開端或徵兆。
人民。
姓。如五代時蜀有萌慮。
發芽。
發生。
```

目前沒把讀音做篩選，不過這樣也感覺蠻有樣子了。

程式碼可以參考 [PR Day 22](https://github.com/MilesChou/namer/pull/8)

## 參考資料

* [萌典][]

[萌典]: https://www.moedict.tw
[Day 20]: /docs/day20.md
