---
Tags:
 - python
 - javascript
 - golang
date: 2016-02-25T10:39:24-05:00
title: Go bridges to Javascript and Python

---

<!--
Preparation for the talk:
* Clear browser history
  * Visit all links in this page, to have quick reference with completion in Chrome
* Change Emacs font size and background color
* Change Terminal font size and background color
* Dry run with a screen size of 1024x768 or something...
* Clean the Desktop
* Temporarily change the background image of my desktop
* Lock preferences: don,t close or lock the screen during presentation.
* Start within a README.md in polyglot/

TODO:
 * add yasnippet for `alert` in GopherJS land
 * add a `poof` yasnippet for `if err != nil { log.Fatalln("Msg", err) }` block.
-->


<!--









     Alexandre Bourget - Data Scientist @ Intel Security
                      @bourgetalexndre

               Golang Montréal - Feb 22nd 2016




Go bridges to Javascript and Python.



We'll demonstrate Go bridges to Javascript (otto, GopherJS) and Python
(gopy). Everything in this session will be live coded from scratch.






-->

> *This is the content of a talk given at Golang Montréal
> [meetup on February 22nd 2016](https://golangmontreal.org/en/events/gomtl-01-go-16-release-party-feb-22nd/),
> at Google Montréal.*

This is a high level introduction to different bridges that exist
between Go and other languages.  It is not meant to be exhaustive nor
does it go very deep in each subject, but I hope it is at least
entertaining.

{{< youtube YmI7Gw4iq3w >}}

<!--more-->

## Javascript

### Otto

The [otto](https://github.com/robertkrimen/otto) library -- not to be
confused with [Otto](https://github.com/hashicorp/otto),
[_the new Vagrant_](https://www.ottoproject.io/) -- is an ECMAScript
5.1 implementation in native Go.  It can thus run javascript code in a
portable way, shipping your single binary with the VM included
(without Cgo or anything of the sort).

Get it with:

    go get github.com/robertkrimen/otto/...

and check
[the docs here](https://godoc.org/github.com/robertkrimen/otto). If
you're interested in the AST parser, check
[the parser package](http://godoc.org/github.com/robertkrimen/otto/parser)

This will also install the `otto` program, so you can run it on the command line like:

    $ otto
    var hello = "world";
    var someMath = 1 + 2 + 3;
    console.log("Hello", hello, someMath);
    ^D
    Hello world 6

This is a simple execution on the command line.  Now let's use it in
our program.  Open up `jsvm/main.go` and write:

```
package main

import (...)

func main() {
	vm := otto.New()
	vm.Set("val1", 123)
	vm.Set("val2", 234)
    out, err := vm.Run(`
ret = val1 + val2;
    `)
	if err != nil {
		log.Fatalln("Error running VM:", err)
	}
	log.Println("Out:", out)

	ret, err := vm.Get("ret")
	val3, err := ret.ToInteger()
	if err != nil {
		log.Fatalln("Can't convert to Integer:", err)
	}

	fmt.Println("Result:", val3)
}
```

and run:

    $ cd jsvm
    $ go run main.go

And you'll see:

    Result: 357
    [INSERT THE REAL THING]

It seems slow because `go run` compiles and runs, so you can try:

    go build -v .
    time ./jsvm

Otto can also pre-compile javascript into a `Script` object, so you
don't need to reparse it on the next run, thereby increasing
performance.

There you go: you can run Javascript inside a Go program, and keep
Go's portability.



### GopherJS


[GopherJS](http://www.gopherjs.org/) is a Go to Javascript compiler.
It supports almost everything, including Goroutines and almost all of
the standard library. You can try things in the
[playground](http://www.gopherjs.org/playground/).

Here we will try it out and show a couple of patterns at work.

Install the library and program with:

    go get -v github.com/gopherjs/gopherjs

and open up `gopherjs/code.go` to insert:

```
package main

// run goimports

func main() {
	//resp, err := http.Get("https://example.com")
	resp, err := http.Get("https://cors-test.appspot.com/test")
	if err != nil {
        println("Failed HTTP query:", err)
	}
	content, err := ioutil.ReadAll(resp.Body)
	if err != nil {
        println("Couldn't read body:", err)
	}

	println("My output", string(content))

	js.Global.Call("alert", "We're done!")
}
```

Then run:

    gopherjs build

Notice the `gopherjs.js` file and its accompanying source maps. This
will allow us to know which line caused errors when debugging. Mapping
directly to our Go code!

Why is the generated `.js` file this big, you ask?  It contains the Go
runtime, management of concurrency, channels, goroutines, chunks of
the standard library you need to your operations, and your code. It's
the same "single-binary" distribution you get for other platforms like
Mac, Windows or Linux.

Then run:

    gopherjs serve

and navigate to
http://localhost:8080/github.com/abourget/polyglot/gopherjs/
... you'll see that the request succeeds !


As an example, here's how you'd modify the DOM in Go:

    gopher := `[insert the content of https://raw.githubusercontent.com/golang-samples/gopher-vector/master/gopher-front.svg here]`
    js.Global.Get("document").Get("body").Set("innerHTML", gopher)

The last crazy thing we could do is insert the `otto` VM and run javascript... in the browser.

Add these to our `main` function:

    otto.New().Run(`console.log("This runs in otto, under gopherjs, in the browser (!!)")`)

You can imagine the `otto` code is quite complex, but it all compiles
to javascript through GopherJS and runs fine in the browser, showing
how sophisticated GopherJS really is.

The generated `gopherjs.js` file is enlarged because the whole
javascript interpreter is in there.


#### GopherJS Conclusion

GopherJS led to the creation of a Web/Desktop/Mobile gaming engine
called [Enj](http://ajhager.com/enj/) continued as
[ENGi](https://github.com/ajhager/engi).  All in Go, but compiles to
javascript, desktop and Android via GoMobile.

<!-- Show the demo at http://ajhager.com/enj/ -->

GopherJS also has a
[list of bindings](https://github.com/gopherjs/gopherjs/wiki/bindings)
to different web frameworks (like Angular, D3, Chrome, jQuery, WebGL),
so you can write nice Go code and JS is called via the GopherJS API. I
find it hilarious.

Some might think it's stupidity, but I think they're wrong.  It might
just be the future of the web.  Read more about [WebAssembly
here](https://medium.com/javascript-scene/what-is-webassembly-the-dawn-of-a-new-era-61256ec5a8f6).

Check out https://github.com/tidwall/digitalrain and its
[live demo](http://tidwall.github.io/digitalrain/) to see an app in
action. It's alllll Go.
[This thread](https://groups.google.com/forum/#!topic/golang-nuts/2xhqPFpzEos)
lists more examples.

Last note, is about
[cmd-go-js](https://github.com/gophergala2016/cmd-go-js) which is a
"feature branch" of the `go` command, that supports GopherJS as an
additional `GOARCH=js`.



## Python, via gopy

[gopy](https://github.com/go-python/gopy) is a deceptively simply way
to write Go code and load it in Python.

Let's first install:

    go get -v github.com/go-python/gopy

Write in `ext/user.go`:

```
package ext

type User struct {
	Username string
	Fullname string
	Password string
}
```

We'll add the `SetPassword(password string)` method that will use a
really powerful Go encryption that we'll
[take from here](http://www.dotnetperls.com/rot13-go).  I loooove
that algorithm, because *encrypts* `cat` as `png`.

```
func (u *User) SetPassword(input string) {
	u.Password = strings.Map(rot13, input)
}
```

Don't forget the content of the `rot13` function below
`SetPassword()`.

Soon, you'll be able to use it from Python !

Run:

    $ gopy bind .

This generates an `ext.so` file that we can load directly with
Python's `import ext`:

    $ python
    >>> import ext
    >>> u1 = ext.User(Fullname="My name")
    >>> u1.Fullname
    'My name'
    >>> u1.SetPassword("this is a very secretive password and cat")
    >>> u1.Password
    'guvf vf n irel frpergvir cnffjbeq naq png'

See ? `png`? I just love it.


#### Breaking the GIL

That's all cool, but we already have a `rot13` implementation in
Python! What's the use ?

Let's try it in Python:

    $ python
    >>> import codecs
    >>> codecs.encode("cat", "rot13")
    'png'

What makes our use case interesting is this:

    >>> import thread
    >>> thread.start_new_thread(lambda: [codecs.encode("have a cat", "rot_13") for x in xrange(10000000)], ())

and watch out your CPUs.  Welcome to the GIL: you can't have more than
one core run Python code.

So let's try an example that makes Go shine: concurrency and
multi-core use -- breaking the dreaded Global Interpreter Lock !

Add to a new `ext/maximus.go` such a thing:

```
package ext

func MaxOutResources() func() {
	done := make(chan struct {})

	f := func() {
		for {
			input := "cat"
			for i := 0; i < 100000; i++ {
				input = strings.Map(rot13, input)
			}

			select {
			case <-done:
				fmt.Println("ffiieeeww!", input)
				return
			default:
			}
		}
	}

	go f()
	go f()
	go f()
	go f()

	return func() {
		close(done)
	}
}
```

This function will spin 4 goroutines, in parallel of the Python
process, doing incredible calculations, and allows the caller to shut
down the operations, tapping into Go's concurrency system.

Now rebuilding and running through python would look like:

    $ gopy bind .
    $ python
    >>> import ext
    >>> f = ext.MaxOutResources()
    >>>

Now watch your CPU max out _all your cores_ !

Call `f()` to stop everything:

    >>> f()
    ffiieeeww! cat
    ffiieeeww! cat
    ffiieeeww! cat
    ffiieeeww! cat

Now you know what 100000 iterations of ROT13 does to your passwords.


#### Dynamically build and load

There is a small python wrapper script around this in
`$GOPATH/src/github.com/go-python/gopy/gopy.py` that you can copy to
our work dir and run with something like:

    python
    >>> import gopy
    >>> ext = gopy.load("github.com/abourget/polyglot/py")
    >>> u1 = ext.User()
    >>> u1.SetPassword("very secure password")
    etc...

This internally runs the `gopy bind` calls and imports it in Python
directly.


#### Conclusion

Now mind you, `gopy` is nowhere near perfect.  There are cases where
it will plainly crash, but there is a case for running Go code within
your Python program, for performance or reusability.


#### Additional readings...

[This post](https://blog.filippo.io/building-python-modules-with-go-1-5/)
is a great starter on the internals of building Go-based python extensions.

Python bridges also include https://github.com/sbinet/go-python : Go
bindings to call into the CPython 2 C-level API.



# Conclusion

I initially wanted to do a talk that covered all bindings to other
languages, but we wouldn't have had time to see any of them.  I leave
links here for your curiosity and many some other time I'll cover
other bridges.

## Other bridges

**Ruby** via https://github.com/ffi/ffi/, see
https://c7.se/go-and-ruby-ffi/ . See also a very experimental Ruby
interpreter https://github.com/grubby/grubby .  There is also
https://github.com/DavidHuie/quartz which involes running a separate
process and transparently communicating via RPC. The last is is a
binding to `mruby` in Go, by none other than HashiCorp's founder,
Mitchell Hashimoto (of Vagrant fame). Check it here:
https://github.com/mitchellh/go-mruby .

**Android** and **iOS** via
[Go Mobile](https://github.com/golang/mobile) which allows you to
write libraries for iOS and Android that you can load in their
respective IDE.  You can also write native application that compile to
native packages. More details on
[the Go Mobile wiki](https://github.com/golang/go/wiki/Mobile)

**Lua** via pure-go implementations of the language. There are actually *two* of them, https://github.com/Shopify/go-lua and https://github.com/yuin/gopher-lua .

## Other oddities

campher: Perl bindings for Go, by Brad Fitzpatrick: https://github.com/bradfitz/campher
llgo: LLVM front-end for Go, in Go: http://llvm.org/svn/llvm-project/llgo/trunk/README.TXT

Attempts at interactive Go REPLs:
[gore](https://github.com/motemen/gore),
[gosh](https://github.com/mkouhei/gosh) and
[go-fish](https://github.com/rocky/go-fish). Attempts at a Go
scripting language interpreter: [igo](https://github.com/sbinet/igo) and
[go-eval](https://github.com/sbinet/go-eval).


### About the presentation

Those interested in my `.emacs.d` dir,
[it is available here](https://github.com/abourget/my.emacs.d).
