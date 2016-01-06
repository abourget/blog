---
Tags:
 - golang
 - goa
 - rest
 - api
date: 2015-12-31T12:00:11-05:00
slug: streaming-files-with-goa-golang
title: Streaming files with goa, the neat Golang code generator
---

I have been working with [goa](http://goa.design) for the past few
days and I must admit I've been impressed.

You define your API in code, it generates your `swagger.json` file as
well as plenty of boilerplate code for you.  It distinguishes nicely
what belongs to code generation and what belongs to your business
logic.

One thing I was trying to do with the current version, was to stream files
in and out (like a file server API).

Here is a sample controller streaming out some `io.Reader` content:

```
func (c *ArtifactController) Get(ctx *app.GetArtifactContext) error {
	file, size, err := storageClient.GetFile(ctx.ArtifactPath)
	if err != nil {
		ctx.Error("storageClient.GetFile", "err", err)
		return err
	}

	ctx.Header().Set("Content-Length", fmt.Sprintf("%d", size))
	ctx.WriteHeader(200)

	_, err = io.Copy(ctx, file)
	return err
}
```

Writing the header before running `io.Copy()` is the trick to make
sure the `goa` controller understand that you've handled the request
properly, and it doesn't need to generate a 500 error.
