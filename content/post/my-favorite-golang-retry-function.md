---
Tags:
 - golang
date: 2016-01-04T14:44:18-05:00
title: "My favorite #golang retry function"

---

Here they are:

```
func retry(attempts int, sleep time.Duration, callback func() error) (err error) {
    for i := 0; ; i++ {
        err = callback()
        if err == nil {
            return
        }

        if i >= (attempts - 1) {
            break
        }

        time.Sleep(sleep)

        log.Println("retrying after error:", err)
    }
    return fmt.Errorf("after %d attempts, last error: %s", attempts, err)
}

func retryDuring(duration time.Duration, sleep time.Duration, callback func() error) (err error) {
	t0 := time.Now()
	i := 0
    for {
		i++

        err = callback()
        if err == nil {
            return
        }

		delta := time.Now().Sub(t0)
        if delta > duration {
			return fmt.Errorf("after %d attempts (during %s), last error: %s", i, delta, err)
        }

        time.Sleep(sleep)

        log.Println("retrying after error:", err)
    }
}
```

Use like this:

```
		var signedContent []byte
		err := retry(5, 2*time.Second, func() (err error) {
			signedContent, err = signFile(unsignedFile, contents)
			return
		})
		if err != nil {
			log.Println(err)
			http.Error(w, err.Error(), 500)
			return
		}
```

I've seen some frameworks and libs to do that.. like
[retry-go](https://github.com/giantswarm/retry-go) and
[backoff](https://github.com/cenkalti/backoff). They're both good, but
a bit overkill for most simple tasks.

I feel it's such a simple function that you'd rather dump in the
project that needs it, tweak the delays and the logging mechanisms,
and be done with it.

<!--more-->
