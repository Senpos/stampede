# Stampede

Prevents cache stampede https://en.wikipedia.org/wiki/Cache_stampede by only running a
single data fetch operation per expired / missing key regardless of number of requests to that key.

## Example 1: HTTP Middleware

```go
import (
	"net/http"
	"time"

	"github.com/go-chi/chi"
	"github.com/go-chi/chi/middleware"
	"github.com/goware/stampede"
)

func main() {
	r := chi.NewRouter()
	r.Use(middleware.Logger)
	r.Use(middleware.Recoverer)

	r.Get("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("index"))
	})

	cached := stampede.Handler(1 * time.Second)

	r.With(cached).Get("/cached", func(w http.ResponseWriter, r *http.Request) {
		// processing..
		time.Sleep(1 * time.Second)

		w.WriteHeader(200)
		w.Write([]byte("...hi"))
	})

	http.ListenAndServe(":3333", r)
}
```


## Example 2: Raw

```go
import (
	"net/http"

	"github.com/goware/stampede"
)

var (
	reqCache = stampede.NewCache(5*time.Second, 10*time.Second)
)

func handler(w http.ResponseWriter, r *http.Request) {	
	data, err := reqCache.Get(r.URL.Path, fetchData)
	if err != nil {	
		w.WriteHeader(503)
		return	
	}

	w.Write(data.([]byte))
}

func fetchData(ctx context.Context) (interface{}, error) {
	// fetch from remote source.. or compute/render..
	data := []byte("some response data")

	return data, nil	
}
```


## LICENSE

MIT