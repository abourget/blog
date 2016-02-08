---
Tags:
 - python
 - javascript
 - golang
date: 2016-01-14T11:04:24-05:00
draft: true
title: It's a polyglot world out there!
alternate_title: Go in a polyglot world: bridges to other languages and platforms
alternate_title: In a polyglot world: Go bridges to other languages and platforms
alternate_title: Overview of Go bridges to other languages and platforms

---

This is the outline of a talk to be given at Golang Montréal.

The world of programming is full of languages, and knowing more than one is often required.

This is why today, I'm going to talk about Go, but interact with as many languages as possible.



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

func main() {
	vm := otto.New()
    vm.Run(`

    `)
}
```


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

Uncomment the `example.com` line, and see the proper `Sprintf` call is
being done, with all formatting. That's the standard Go library.

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
    canvas.Circle(125, 125, 100, "fill:#28262C;stroke:#5BA642;stroke-width:5px")
    canvas.Text(125, 135, "GopherJS!", "text-anchor:middle;font-size:36px;fill:#EB5633")
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
just be the future of the web.  Read more about
[WebAssembly here](https://medium.com/javascript-scene/what-is-webassembly-the-dawn-of-a-new-era-61256ec5a8f6).



## Python

### gopy


https://github.com/go-python/gopy

Build your binaries before, or after

Intro to building .so files

  https://blog.filippo.io/building-python-modules-with-go-1-5/


### go-python

Python bridges also include https://github.com/sbinet/go-python : Go
bindings to call into the CPython 2 C-level API.


Ruby FFI
--------

https://c7.se/go-and-ruby-ffi/

would that make sense ?

  Build for many platforms..  demo under Windows (through wine)

[Check: https://github.com/mitchellh/go-mruby]


Lua VM
------

https://github.com/Shopify/go-lua
https://github.com/yuin/gopher-lua



Write your dynamic/scripting language in Go, and you immediately gain
embeddability and cross-platform portability ! How fantastic!


Go Mobile
---------

https://github.com/golang/mobile

http://www.sajalkayan.com/post/go-android-binary.html?


Other oddities
--------------

campher: Perl bindings for Go, by Brad Fitzpatrick: https://github.com/bradfitz/campher
llgo: LLVM front-end for Go, in Go: http://llvm.org/svn/llvm-project/llgo/trunk/README.TXT

Attempts at interactive Go REPLs:
[gore](https://github.com/motemen/gore),
[gosh](https://github.com/mkouhei/gosh) and
[go-fish](https://github.com/rocky/go-fish). Attempts at a Go
scripting language interpreter: [igo](https://github.com/sbinet/igo) and
[go-eval](https://github.com/sbinet/go-eval).

<-- maybe a quick demo of `gore`? too off-topic ? -->
