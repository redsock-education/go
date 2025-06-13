Функция в go это _тип_ 

Типичный формат применения этой ****

```go
package clients

import (
	"time"
)

type ApiCaller struct {
	urlPath string
	timeout time.Time
	// and more and more
}

var opt func(caller *ApiCaller)

```