# modules

This document covers userland packages you can use to manipulate and consume
streams.

The core stream APIs are very tricky to implement correctly, so generally
speaking it's much easier and faster to use these libraries.

---
# through2

create transform streams with a function instead of subclassing

``` js
var through = require('through2')
process.stdin
  .pipe(through(write, end))
  .pipe(process.stdout)

function write (buf, enc, next) {
  next(null, buf.toString().toUpperCase())
}
function end (next) {
  next()
}
```

---
# through2: this

You might want to call `this.push()` instead of `next()`,
but make sure to also call `next()`.

``` js
var through = require('through2')
process.stdin
  .pipe(through(write, end))
  .pipe(process.stdout)

function write (buf, enc, next) {
  this.push(buf.toString().toUpperCase())
  next()
}
function end (next) {
  this.push('\nEND\n')
  next()
}
```

---
# through2: defaults

```
through(opts={}, write, end)
```

You can omit write and end arguments, these are the defaults:

``` js
function write (buf, enc, next) {
  next(null, buf)
}
function end (next) {
  next()
}
```

---
# through2: obj

create a through stream in objectMode:

```
through.obj(opts={}, write, end)
```

---
# split2

split input on newlines

This program counts the number of lines of input, like `wc -l`:

``` js
var split = require('split2')
var through = require('through2')

var count = 0
process.stdin.pipe(split())
  .pipe(through(write, end))

function write (buf, enc, next) {
  count++
  next()
}
function end () {
  console.log(count)
}
```

---
# split2

A note about split2:

In each line, the trailing `'\n'` is removed.

---
# split2

You can give `split()` a custom string or regex to split on:

This program splits on all whitespace:

``` js
var split = require('split2')
process.stdin.pipe(split(/\s+/))
  .pipe(through(function (buf, enc, next) {
    
    next()
  }))
```

---
# split2

You can also give `split()` a function to parse on:

``` js
var split = require('split2')
var sp = split(JSON.parse)
process.stdin.pipe(sp)
```

---
# concat-stream

buffer up stream contents into a single buffer

---
# concat-stream

``` js
var concat = require('concat-stream')
var qs = require('querystring')
var http = require('http')
var server = http.createServer(function (req, res) {
  req.pipe(concat(function (body) {
    var params = qs.parse(body.toString())
    console.log('params=', params)
    res.end('ok\n')
  }))
})
server.listen(5000)
```

---
# concat-stream

```
$ node server.js &
$ node server.js &
[1] 19021
$ curl -X POST -d hello=world -d beep=boop http://localhost:5000
params= { hello: 'world', beep: 'boop' }
ok
$ kill %1
```

---
# collect-stream

collect a stream's output into a single buffer

for object streams, collect output into an array of objects

``` js
var collect = require('collect-stream')
var split = require('split2')

var sp = process.stdin.pipe(split(JSON.parse))
collect(sp, function (err, rows) {
  if (err) console.error(err)
  else console.log(rows)
})
```

This module is very useful for unit tests.

---
# from2

create a readable stream with a pull function

``` js
var from = require('from2')
var messages = [ 'hello', ' world\n', null ]

from(function (size, next) {
  next(null, messages.shift())
}).pipe(process.stdout)
```

---
# to2

create a writable stream with a write and flush function

``` js
var to = require('to2')
var split = require('split2')

process.stdin.pipe(split()).pipe(to(function (buf, next) {
  console.log(buf.length)
  next()
}))
```

---
# duplexify

``` js
var duplexify = require('duplexify')
var d = duplexify()

d.setReadable(...)
d.setWritable(...)
```

---
# pump

``` js
var pump = require('pump')

pump(stream1, stream2, stream3, ...)
```

---
# pumpify

``` js
var pump = require('pumpify')

var stream = pump(stream1, stream2, stream3, ...)
```

---
# end-of-stream

reliably detect when a stream is finished

``` js
var onend = require('end-of-stream')
var net = require('net')

var server = net.createServer(function (stream) {
  var iv = setInterval(function () {
    stream.write(Date.now() + '\n')
  }, 1000)
  onend(stream, function () {
    clearInterval(iv)
  })
})
server.listen(5000)
```
