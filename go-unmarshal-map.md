---
title: "go 语言 unmarshal 到 map"
date: 2023-03-01T17:04:04+08:00
draft: false
---

测试代码如下：
```golang
func Test() error {
	mp := make(map[string]string)
	mp["e1"] = "e1"
	mp["e2"] = "e2"
	bt := `
		{"e3":"e3","e2":"e1000"}
	`
	err := json.Unmarshal([]byte(bt), &mp)
	if err != nil {
		return err
	}
	log.Println("map is:", mp)
	return nil
}
```
结果如下：
```bash
2023/03/01 16:55:15 map is: map[e1:e1 e2:e1000 e3:e3]
```

Unmarshal 函数的注释如下：
```golang
// To unmarshal a JSON object into a map, Unmarshal first establishes a map to
// use. If the map is nil, Unmarshal allocates a new map. Otherwise Unmarshal
// reuses the existing map, keeping existing entries. Unmarshal then stores
// key-value pairs from the JSON object into the map. The map's key type must
// either be any string type, an integer, implement json.Unmarshaler, or
// implement encoding.TextUnmarshaler.
```

```golang
// To unmarshal a JSON array into a slice, Unmarshal resets the slice length
// to zero and then appends each element to the slice.
// As a special case, to unmarshal an empty JSON array into a slice,
// Unmarshal replaces the slice with a new empty slice.
```
为什么不像 slice 那样清空呢？可能是因为 slice 有一个很方便的操作是 reset length，而 map 没有 clear 函数？
