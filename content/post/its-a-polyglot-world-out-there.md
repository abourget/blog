---
Tags:
 - python
 - javascript
 - golang
date: 2016-01-14T11:04:24-05:00
draft: true
title: It's a polyglot world out there!
#alternate_title: Go in a polyglot world: bridges to other languages and platforms
#alternate_title: In a polyglot world: Go bridges to other languages and platforms
#alternate_title: Overview of Go bridges to other languages and platforms

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

TODO:
 * add yasnippet for `alert` in GopherJS land
 * add a `poof` yasnippet for `if err != nil { log.Fatalln("Msg", err) }` block.
-->

> This is the contents of a talk given at Golang Montréal
> [meetup on February 22nd 2016](https://golangmontreal.org/en/events/gomtl-01-go-16-release-party-feb-22nd/),
> at Google Montréal.

This is a high level introduction to different bridges that exist
between Go and other languages.  It is not meant to be exhaustive nor
does it go very deep in each subject, but I hope it is at least
entertaining.


## Javascript

### Otto

The [otto](https://github.com/robertkrimen/otto) library -- not to be
confused with [Otto](https://github.com/hashicorp/otto),
[_the new Vagrant_](https://www.ottoproject.io/) -- is an ECMAScript
5.1 implementation in native Go.  It can thus run javascript code in a
portable way, shipping your single binary with the VM included
(without Cgo or anything of the sort).

Get it with `go get github.com/robertkrimen/otto/...` and check
[the docs here](https://godoc.org/github.com/robertkrimen/otto). If
you're interested in the AST parser, check
[the parser package](http://godoc.org/github.com/robertkrimen/otto/parser)

Let's start by writing some Javascript code and running it through
`otto`. Drop this in `hello.js`:

```
var hello = "world";
var someMath = 1 + 2 + 3;
console.log("hello", hello, someMath);
```

and run:

    otto hello.js

This is a simple execution on the command line.  Now let's use it in
our program.  Write `otto.go` somewhere:

```
package main

import (...)

func main() {
	vm := otto.New()
	vm.Set("val1", 123)
	vm.Set("val2", 234)
    _, err := vm.Run(`
ret = val1 + val2;
    `)
	if err != nil {
		log.Fatalln("Error running VM:", err)
	}

	ret, err := vm.Get("ret")
	val3, err := ret.ToInteger()
	if err != nil {
		log.Fatalln("Can't convert to Integer:", err)
	}

	fmt.Println("Result:", val3)
}
```

and run it with:

    go run otto.go

And you should see:

    Result: 357

It seems slow because `go run` compiles and runs, so you can try:

    go build -v .
    time ./otto

And there you for embedded JS !



### GopherJS


[GopherJS](http://www.gopherjs.org/) is a Go to Javascript compiler.
It supports almost everything, including Goroutines and almost all of
the standard library. You can try things in the
[playground](http://www.gopherjs.org/playground/).

Here we will try it out and show a couple of patterns at work.

Install with `go get -v github.com/gopherjs/gopherjs`, and put in `code.go` somewhere:

```go
package main

// run goimports

func main() {
	//resp, err := http.Get("https://example.com")
	resp, err := http.Get("https://cors-test.appspot.com/test")
	if err != nil {
		js.Global.Call("alert", fmt.Sprintf("Failed HTTP query: %s", err))
	}
	content, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		js.Global.Call("alert", "Couldn't read body", err)
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

Why is it this big you ask?  It contains the Go runtime, management of
concurrency, channels, goroutines, chunks of the standard library you
need to your operations, and your code. It's the same "single-binary"
distribution you get for other platforms like Mac, Windows or Linux.

Then run:

    gopherjs serve

and navigate to
http://localhost:8080/github.com/abourget/polyglot/gopherjs/

You'll see that the request succeeds !

Uncomment the `...Get("...example.com/")` line, and see the proper
`Sprintf` call is being done, with all formatting. That's the standard
Go library.

Let's try to unmarshal the JSON with static typing in javascript:

```
	var status struct {
		Status string
	}

	json.Unmarshal(content, &status)

	println("Success !", status.Status)

```

Add this near the end of the previous function.  That runs the Go JSON
marshalling magic, happening in your browser with all of its
reflection glory.

Let's get a bit more crazy and embed `otto` in your browser. Yes, that
means having an ECMAScript virtual machine, written in pure Go,
compiled to Javascript, running in the browser, which will run yet
some more javascript code for your:

```
func launchVM() {
	vm := otto.New()
	val, err := vm.Run(`
      abc = 2 + 2;
      console.log("The value of abc is " + abc); // 4
      return "world";
    `)
	if err != nil {
		println("got an error running the otto VM", err)
	}
	println("Got from inside the otto VM:", val)
}
```

and call that from your `main()` function.

You can imagine the `otto` code is quite complex, but it all compiles
to javascript through GopherJS and runs fine in the browser.  You most
probably will never run code like that in production however,
hopefully :)

#### Generating SVG

[This article](http://www.gopherjs.org/blog/2014/10/29/svgo/) shows
how to use the Go library [SVGo](https://github.com/ajstarks/svgo) in
a very terse and neat way. Let's try it here:

Get the dependency with `go get github.com/ajstarks/svgo`. Then add a
new function to main and define it as such:

```
func genSVG() {
    var output bytes.Buffer

    canvas := svg.New(&output)
    canvas.Start(250, 250)
    canvas.Circle(125, 125, 100, "fill:#28262C; stroke:#5BA642; stroke-width:5px")
    canvas.Text(125, 135, "GopherJS!", "text-anchor:middle; font-size:36px; fill:#EB5633")
    canvas.End()

    time.Sleep(1 * time.Second)

    js.Global.Get("document").Get("body").Set("innerHTML", output.String())
}
```

This shows a nice generation library being used directly in the
browser, with no code change, and can run on all platforms natively,
shipped as a single binary, and as we're going to see, we'll be able
to run such code many other places.

Continue on...


<!--

Would we try this one too ?

#### Image manipulations

https://github.com/nfnt/resize

-->


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

Last note, is about
[cmd-go-js](https://github.com/gophergala2016/cmd-go-js) which is a
"feature branch" of the `go` command, that supports GopherJS as an
additional `GOARCH=js`.


## Python

### gopy

[gopy](https://github.com/go-python/gopy) is a deceptively simply way
to write Go code and load it in Python.

Let's first install:

    go get -v github.com/go-python/gopy

Let's try it out by writing some code in `user.go`, in a clean directory:

    mkdir -p $GOPATH/src/github.com/abourget/polyglot/ext; cd !$

then in `lib.go`:

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
[take from here](http://www.dotnetperls.com/rot13-go).  I just love
that algorithm, because using this very method, `cat` get encrypted to
`png` !

```
func (u *User) SetPassword(input string) {
	u.Password = strings.Map(rot13, input)
}
```

Don't forget the content of the `rot13` function below
`SetPassword()`.

Soon, you,ll be able to use it from Python !

Run:

    $ gopy bind .

This generates a `.so` file that we can load directly with Python's
`import ext`:

    $ python
    >>> import ext
    >>> u1 = ext.User()
    >>> u1.Fullname = "My name"
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
    >>> import thread
    >>> thread.start_new_thread(lambda: [codecs.encode("have a cat", "rot_13") for x in xrange(10000000)])

and watch out your CPUs.  Welcome to the GIL: you can't have more than
one core run Python code.

So let's try an example that makes Go shine: concurrency and
multi-core use -- breaking the dreaded Global Interpreter Lock !

Add to a new `maximus.go` such a thing:

```
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
down the operations, tapping into the sweet Go concurrency system.

Now rebuilding and running through python would look like:

    $ gopy bind .
    $ python
    >>> import ext
    >>> f = ext.MaxOutResources()
    >>>

Now watch your CPU max out _all your cores_ !

Call `f()` to stop everything:

```
    >>> f()
    ffiieeeww! cat
    ffiieeeww! cat
    ffiieeeww! cat
    ffiieeeww! cat
```

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



## Ruby FFI and c-shared build mode

[ffi](https://github.com/ffi/ffi/) is [Foreign Function
Interface](https://en.wikipedia.org/wiki/Foreign_function_interface)
for Ruby.  It allows you to write native code, in some other language,
and to/from it from Ruby.

Today we're interested in Go, so let's use the `ffi` gem to call
to/from Go

<!-- https://c7.se/go-and-ruby-ffi/ -->

First, we install the `ffi` gem:

    gem install ffi

    # On Linux, you might need `ruby1.9.1-dev` or `ruby2.0-dev` installed

Now let's define `libsum.go` (ripped from https://c7.se/go-and-ruby-ffi/):

    package main

    import "C"

    //export add
    func add(a, b int) int {
    	return a + b
    }

    func main() {}

we'll build it with:

    go build -buildmode=c-shared -o libsum.so libsum.go

and we'll create a `sum.rb` file with:

    require 'ffi'

    module Sum
      extend FFI::Library
      ffi_lib './libsum.so'
      attach_function :add, [:int, :int], :int
    end

    puts Sum.add(15, 27)

Let's try to run it with `ruby` now:

    ruby sum.rb

Now notice there is a SEGFAULT there, It seems there's an issue,
either with FFI or with my system configuration. I've reported the
issue, but YMMV.

Now let's add a twist.  What if we could interpret ruby code directly
in Go, like we did with Javascript ?

The `grubby` folks made a Ruby interpreter, in native Go.  Bear with
me here though, **it's pretty rough!**.

Get it:

    go get github.com/grubby/grubby

It comes with programs that replace `irb` and `ruby` with grubby's
implementation. Find it under `main/irb` and `main/ruby`.

Let's add a function to our ruby code, and then call it through FFI:

    module Sum
      extend FFI::Library
      ffi_lib './libsum.so'
      attach_function :add, [:int, :int], :int
      attach_function :runCode, [:string], :string
    end

    puts Sum.runCode(<<-DOC

    require 'fileutils'

    puts "This runs inside grubby and adds stuff:", 5 + 7

    return FileUtils.pwd()

    DOC
    )

Add something like this to `libsum.go`:

    //export runCode
    func runCode(codePtr *C.char) string {
    	code := C.GoString(codePtr)

    	vm := vm.NewVM(os.Getenv("GOPATH")+"/src/github.com/grubby/grubby", "my VM")
    	defer vm.Exit()

    	//fmt.Printf("Running code: %s\n", code)
    	res, err := vm.Run(code)
    	if err != nil {
    		return fmt.Sprintf("error: %s", err)
    	}

    	if res != nil {
    		fmt.Println("Class:", res.Class().Name())
    		fmt.Println("Result:", res.String())
    		return res.String()
    	}

    	return "done"
    }

You'll need to symlink the `grubby` path to `~/.grubby` for the `lib`
content to load through `require` calls:

    ln -s $GOPATH/src/github.com/grubby/grubby ~/.grubby

and let's rebuild and run the `sum.rb` file again:

    go build -buildmode=c-shared -o libsum.so libsum.go && ruby sum.rb

And we'll see:

    This runs inside grubby and adds stuff:
    12
    Class: String
    Result: /home/abourget/go/src/github.com/abourget/polyglot/ffi
    /home/abourget/go/src/github.com/abourget/polyglot/ffi
    We're done

You do need to know that `grubby` is *very* experimental, however the
FFI stuff should be very useable using Cgo bindings.

#### Other ways

There are (at least) two other ways to interact with Ruby from/to Go:

* The first, https://github.com/DavidHuie/quartz, involves running a
  separate process and communicating through an RPC protocol.

* The second, is a binding to `mruby` in Go, by none other than
  HashiCorp's founder, Mitchell Hashimoto (of Vagrant fame). Check it
  here: https://github.com/mitchellh/go-mruby



## Go Mobile

[Go Mobile](https://github.com/golang/mobile) allows you to write
libraries for iOS and Android that you can load in their respective
IDE.  You can also write native application that compile to native
packages.  The APIs available for the native applications do not
include any of the GUI stuff at all at this time, except OpenGL that
works on both platforms.  Do remember that all this is early stage.

Get it with:

    go get -v golang.org/x/mobile/cmd/gomobile

We'll need to install the `gomobile` toolchain for Android:

    gomobile init

I like how most things in Go are one-liners, with almost no output.

Create a new directory and go into it:

    mkdir -p $GOPATH/src/github.com/abourget/polyglot/mobile; cd !$

Let's copy the example app we find at https://godoc.org/golang.org/x/mobile/app into our `main.go` and start from there:

```
package main

func main() {
	app.Main(func(a app.App) {
		for e := range a.Events() {
			switch e := a.Filter(e).(type) {
			case lifecycle.Event:
				// ...
			case paint.Event:
				log.Print("Call OpenGL here.")
				a.Publish()
			}
		}
	})
}
```

Now we'll tweak our program slightly to make a POST request when the
state of the app changes:

```
			switch e := a.Filter(e).(type) {
			case lifecycle.Event:
				_, _ = http.Post(fmt.Sprintf("http://192.168.86.151:8888?from=%s&to=%s", e.From, e.To), "", nil)
			...
```

Let's use this simple server to see what we receive (in
`serve/main.go` or something):

```
func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		cnt, _ := httputil.DumpRequest(r, false)
		os.Stdout.Write(cnt)
	})
	fmt.Println("Listening")
	http.ListenAndServe(":8888", nil)
}
```

I'm going to use `genymotion` to simulate an Android device. The goal
is to have the program injected in the VM as fast as possible.

Let's compile that as an `.apk` and see what happens:

    gomobile install

Boom, it's installed directly to the device. This one will be called
`Mobile` as its in the `mobile` program package.

Notice in the `adb logcat`, the `log.Print` call being
displayed. Notice also the state changes. It's very barebone, but
we've done much with very little until now.

More details can be found on
[the Go Mobile wiki](https://github.com/golang/go/wiki/Mobile).


## Cross-platform cross-compilation

I would have forgotten a major piece if I hadn't touched this last
subject.

One of Go's major feature is the ability to cross-compile to any
platform, from any platform.  On Linux, build for Windows, on Windows,
build for Mac OS X, etc..

And it's the simplicity of it all that makes it the most attractive:

    GOOS=windows GOARCH=386 go build -v .

for a given program.  Then try it through `wine` or look with `file`
what type of executable you have at hand:

    file program.exe
    wine program.exe

Go uses _build tags_ to decide whether a file gets included in the
compilation process or not.

Any filename ending with `_windows.go` will only be compiled under the
`windows` GOOS target, same with `_darwin.go` or `_linux.go`.

Additionally, you can use
[build constraints](https://golang.org/pkg/go/build/#hdr-Build_Constraints)
to specify such tags inside the file. Example:

    // +build !windows

meaning that we want it on all platforms except Windows.



## Lua VM

I was originally going to make a demonstration of the different Go
implementations of Lua, but at the next meetup, we will have one of
the main authors of `go-lua` present his work directly.

As a reference:

* https://github.com/Shopify/go-lua
* https://github.com/yuin/gopher-lua



## Conclusions

Note how when you write your dynamic/scripting language in Go and you
immediately gain embeddability and cross-platform portability ! How
fantastic!  This is a challenge to C, where toolchain issues prevent
true cross-platform portability (at least misses the *easy* part).



### Other oddities

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

<-- maybe a quick demo of `gore`? too off-topic ? -->
