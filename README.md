This is an [Tantivy](https://github.com/quickwit-oss/tantivy) full-text search engine starter template that compiles to [WASIX](https://wasix.org).

## Getting started

## Creating the index:  `new`
 
Let's create a directory in which your index will be stored.

```bash
    # create the directory
    mkdir wikipedia-index
```

We will now initialize the index and create its schema.
The [schema](https://quickwit-oss.github.io/tantivy/tantivy/schema/index.html) defines
the list of your fields, and for each field:
- its name 
- its type, currently `u64`, `i64` or `str`
- how it should be indexed.

You can find more information about the latter on 
[tantivy's schema documentation page](https://quickwit-oss.github.io/tantivy/tantivy/schema/index.html)

In our case, our documents will contain
* a title
* a body 
* a url

We want the title and the body to be tokenized and indexed. We also want 
to add the term frequency and term positions to our index.

Running `tantivy new` will start a wizard that will help you
define the schema of the new index.

Like all the other commands of `tantivy`, you will have to 
pass it your index directory via the `-i` or `--index`
parameter as follows:

```bash
$ wasmer run wasmer/tantivy --mapdir /wikipedia-index:wikipedia-index -- new -i wikipedia-index
```

Answer the questions as follows:

```none
Creating new index 
First define its schema! 



New field name  ? title
Choose Field Type (Text/u64/i64/f64/Date/Facet/Bytes/Json/bool/IpAddr) ? Text
Should the field be stored (Y/N) ? Y
Should the field be fast (Y/N) ? Y
Should the field be indexed (Y/N) ? Y
Should the term be tokenized? (Y/N) ? Y
Should the term frequencies (per doc) be in the index (Y/N) ? Y
Should the term positions (per doc) be in the index (Y/N) ? Y
Add another field (Y/N) ? Y



New field name  ? body
Choose Field Type (Text/u64/i64/f64/Date/Facet/Bytes/Json/bool/IpAddr) ? Text
Should the field be stored (Y/N) ? Y
Should the field be fast (Y/N) ? Y
Should the field be indexed (Y/N) ? Y
Should the term be tokenized? (Y/N) ? Y
Should the term frequencies (per doc) be in the index (Y/N) ? Y
Should the term positions (per doc) be in the index (Y/N) ? Y
Add another field (Y/N) ? Y



New field name  ? url
Choose Field Type (Text/u64/i64/f64/Date/Facet/Bytes/Json/bool/IpAddr) ? Text
Should the field be stored (Y/N) ? Y
Should the field be fast (Y/N) ? N
Should the field be indexed (Y/N) ? N
Add another field (Y/N) ? N

[
  {
    "name": "title",
    "type": "text",
    "options": {
      "indexing": {
        "record": "position",
        "fieldnorms": true,
        "tokenizer": "en_stem"
      },
      "stored": true,
      "fast": true
    }
  },
  {
    "name": "body",
    "type": "text",
    "options": {
      "indexing": {
        "record": "position",
        "fieldnorms": true,
        "tokenizer": "en_stem"
      },
      "stored": true,
      "fast": true
    }
  },
  {
    "name": "url",
    "type": "text",
    "options": {
      "stored": true,
      "fast": false
    }
  }
]
```

After the wizard has finished, a `meta.json` should exist in `wikipedia-index/meta.json`.
It is a fairly human readable JSON, so you can check its content.

It contains two sections:
- segments (currently empty, but we will change that soon)
- schema 

# Indexing the document: `index`

Tantivy's `index` command offers a way to index a json file.
The file must contain one JSON object per line.
The structure of this JSON object must match that of our schema definition.

```json
    {"body": "some text", "title": "some title", "url": "http://somedomain.com"}
```

For this tutorial, you can download a corpus with the 5 million+ English Wikipedia articles in the right format here: [wiki-articles.json (2.34 GB)](https://www.dropbox.com/s/wwnfnu441w1ec9p/wiki-articles.json.bz2?dl=0).
Make sure to decompress the file. Also, you can avoid this if you have `bzcat` installed so that you can read it compressed.

```bash
    bunzip2 wiki-articles.json.bz2
```

If you are in a rush you can [download 100 articles in the right format here (11 MB)](http://fulmicoton.com/tantivy-files/wiki-articles-1000.json).

The `index` command will index your document.
By default it will use as 3 thread, each with a buffer size of 1GB split a
across these threads.

NOTE: Put the wikipedia articles file that you have downloaded in the `wikipedia-index` directory for now so the wasm code can access it.

```bash
$ wasmer run wasmer/tantivy --mapdir /wikipedia-index:wikipedia-index -- index -f /wikipedia-index/wiki-articles-1000.json -i /wikipedia-index

Commit succeed, docstamp at 10001
Waiting for merging threads
Total Nowait Merge: 2.03 Mb/s
Total Wait Merge: 2.03 Mb/s
Terminated successfully
```

You can change the number of threads by passing it the `-t` parameter, and the total
buffer size used by the threads heap by using the `-m`. Note that tantivy's memory usage
is greater than just this buffer size parameter.

While tantivy is indexing, you can peek at the index directory to check what is happening.

```bash
    ls ./wikipedia-index
```

The main file is `meta.json`.

You should also see a lot of files with a UUID as filename, and different extensions.
Our index is in fact divided in segments. Each segment acts as an individual smaller index.
Its name is simply a uuid. 

If you decided to index the complete wikipedia, you may also see some of these files disappear.
Having too many segments can hurt search performance, so tantivy actually automatically starts
merging segments. 

NOTE: You can now remove the wikipedia articles json file that you've downloaded since it's no longer needed.

# Serve the search index: `serve`

Tantivy's cli also embeds a search server.
Now that the index files are ready, you can run the package with the following command.

```bash
$ wasmer run --net --mapdir /index:wikipedia-index .
listening on http://localhost:3000
```

By default, it will serve on port `3000`.

Then you can start querying for different terms such as:
```bash
$ curl http://localhost:3000/api/?q=tomato&nhits=20
```

## Deploy on Wasmer Edge

The easiest way to deploy your Tantivy app is to use the [Wasmer Edge](https://wasmer.io/products/edge).

```bash
wasmer deploy
```