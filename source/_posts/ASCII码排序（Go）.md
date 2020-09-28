---
title: ASCII码排序（Go）
date: 2020-09-28 16:53:14
categories:
  - Golang
  - 算法
tag:
  - 冒泡排序
---

`Problem Description` ：	输入三个字符后，按各字符的ASCII码从小到大的顺序输出这三个字符。
`Input` ：					输入数据有多组，每组占一行，有三个字符组成，之间无空格。
`Output` ：				对于每组输入数据，输出一行，字符中间用一个空格分开。
`Sample Input` ：			qwe/asd/zxc
`Sample Output` ：			e q w/a d s/c x z

<!--more-->


```go
func sort(arr []int32) []int32 {
	for i := 0; i <= len(arr); i++ {
		for j := i + 1; j <= len(arr)-1; j++ {
			if arr[i] > arr[j] {
				temp := arr[j]
				arr[j] = arr[i]
				arr[i] = temp
			}
		}
	}
	return arr
}

func main() {
	var input string
	fmt.Printf("请输入三个英文字符：")
	_, _ = fmt.Scan(&input)
	var asclls = []int32(input)
	result := sort(asclls)
	for _, v := range result {
		fmt.Printf("%v ", string(v))
	}
}
```