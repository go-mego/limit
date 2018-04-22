# Limit [![GoDoc](https://godoc.org/github.com/go-mego/limit?status.svg)](https://godoc.org/github.com/go-mego/limit)

Limit 是用來協助伺服端建立限制的安全機制套件，例如：流量、同時請求數、內容大小。

# 索引

* [安裝方式](#安裝方式)
* [使用方式](#使用方式)
    * [請求大小](#請求大小)
	* [配額限制](#配額限制)
	* [同時請求數](#同時請求數)

# 安裝方式

打開終端機並且透過 `go get` 安裝此套件即可。

```bash
$ go get github.com/go-mego/limit
```

# 使用方式

Limit 有不同的使用方式。

## 請求大小

為了避免某個請求上傳了太大且不必要的內容，可以透過 `size.Limiter` 來加以限制。

```go
package main

import (
	"github.com/go-mego/limit/size"
	"github.com/go-mego/mego"
)

func main() {
	m := mego.New()
	// 將大小限制器用於全域中介軟體時，
	// 會強制要求所有路由的請求大小必須低於這個指定的 KB 大小。
	m.Use(size.Limiter(&size.Options{
		// 允許的最大請求大小。
		Limit: 3 * size.MB,
	}))
	m.Run()
}
```

大小限制器同時也能夠用在單一路由上。

```go
func main() {
	m := mego.New()
	// 大小限制器同時也能夠只用在單一個路由上，這能夠讓不同路由有著不同的大小限制。
	m.POST("/", size.Limiter(&size.Options{
		// ...
	}), func() {
		// ...
	})
	m.Run()
}
```

## 配額限制

如果你希望限制客戶端在單個方法的某個時間內僅能呼叫數次，透過 `rate.Limiter` 就能達成。

```go
package main

import (
	"time"

	"github.com/go-mego/limit/rate"
	"github.com/go-mego/mego"
)

func main() {
	m := mego.New()
	// 將流量限制器用於全域中介軟體時，
	// 會強制單個客戶端在所有路由請求時，不得在指定時間內大於一定的請求量。
	m.Use(rate.Limiter(&rate.Options{
		// 逾期週期，過了這段時間會重設限制。
		Period: 1 * time.Hour,
		// 週期內所允許的最大的請求次數。
		Limit: 100,
	}))
	m.Run()
}
```

配額限制器同時也能夠用在單一路由上。

```go
func main() {
	m := mego.New()
	// 流量限制器同時也能夠套用在單一路由上，這讓你能夠在不同路由有著不同的限制。
	m.GET("/", rate.Limiter(&rate.Options{
		// ...
	}), func() {
		// ...
	})
	m.Run()
}
```

## 同時請求數

為了避免某個方法特別繁忙，透過 `request.Limiter` 可以限制單個方法的最大同時連線數。

```go
package main

import (
	"github.com/go-mego/limit/request"
	"github.com/go-mego/mego"
)

func main() {
	m := mego.New()
	// 將請求限制器用於全域中介軟體時，
	// 會強制要求所有路由的請求同時數必須低於這個數字。
	m.Use(request.Limiter(&request.Options{
		// 最大的同時連線數。
		Limit: 100,
	}))
	m.Run()
}
```

請求限制器同時也能夠用在單一路由上。

```go
func main() {
	m := mego.New()
	// 請求限制器同時也能夠只用在單一個路由上，這能夠讓不同路由有著不同的同時請求數限制。
	m.GET("/", request.Limiter(&request.Options{
		// ...
	}), func() {
		// ...
	})
	m.Run()
}
```



