---
Tags:
 - golang
date: 2016-01-04T14:44:18-05:00
title: My favorite Golang retry function

---

Here it is:

```
func retry(attempts int, callback func() error) (err error) {
	for i := 0; ; i++ {
		err = callback()
		if err == nil {
			return nil
		}

		if i >= (attempts - 1) {
			break
		}

		time.Sleep(2 * time.Second)

		log.Println("retrying...")
	}
	return fmt.Errorf("after %d attempts, last error: %s", attempts, err)
}
```

Use like this:

```
		var valuableContent []byte
		err := retry(5, func() error {
            var err error
			valuableContent, err = signFile(unsignedFile, contents)
			if err != nil {
				return err
			}
			return nil
		})
		if err != nil {
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
and be done.

I license this to your under the MIT license.

<!--more-->
