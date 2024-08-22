This is an [Tantivy](https://github.com/quickwit-oss/tantivy) full-text search engine starter template that compiles to [WASIX](https://wasix.org).

## Getting started
You can run the server using the following command:
```bash
$ wasmer run --net --mapdir /index:wikipedia-index .
listening on http://localhost:3000
```

Then you can start querying for different terms such as:
```bash
$ curl http://localhost:3000/api/?q=tomato&nhits=20
```

## Deploy on Wasmer Edge

The easiest way to deploy your Tantivy app is to use the [Wasmer Edge](https://wasmer.io/products/edge).

```bash
wasmer deploy
```