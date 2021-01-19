# 쓰레드와 코루틴의 차이

- 결정적으로 OS의 개입 유무가 중요함

## 쓰레드

![thread](https://aaronryu.github.io/2019/05/27/coroutine-and-thread/context-switch-between-threads.png)

## 코루틴

![coroutine](https://aaronryu.github.io/2019/05/27/coroutine-and-thread/no-context-switch-between-coroutines.png)

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func f(n int) {
	for i := 0; i < 10; i++ {
		fmt.Println(n, ":", i)
		amt := time.Duration(rand.Intn(250))
		time.Sleep(time.Millisecond * amt)
	}
}
func main() {
	for i := 0; i < 10; i++ {
		go f(i)
	}
	var input string
	fmt.Scanln(&input)
}
```

## 타임라인

![timeline](https://aaronryu.github.io/2019/05/27/coroutine-and-thread/concurrency-progress-bar-thread-and-coroutine.png)

- 모든 Task를 Thread A 에서 실행하게 하면 context switching 이 아예 없게 만들수도 있음

[참고링크](https://aaronryu.github.io/2019/05/27/coroutine-and-thread/)
