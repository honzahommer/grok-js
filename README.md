# grok-js
> Fork of https://github.com/Beh01der/node-grok

[![npm version](https://img.shields.io/npm/v/grok-js.svg)](https://npmjs.com/grok-js)
[![Build Status](https://img.shields.io/travis/honzahommer/grok-js.svg?branch=master)](https://travis-ci.org/honzahommer/grok-js)
[![npm downloads](https://img.shields.io/npm/dm/grok-js.svg)](https://npmjs.com/grok-js)
[![License](https://img.shields.io/npm/l/grok-js.svg)](https://github.com/honzahommer/grok-js/blob/master/LICENSE)
[![Greenkeeper badge](https://badges.greenkeeper.io/honzahommer/grok-js.svg)](https://greenkeeper.io/)

This library is inspired by logstash grok filter but it's not a port of it.

More details about usage and implementation here https://memz.co/parsing-log-files-node-js-regex-grok/

This is a templating library that helps reusing existing regular expressions and constructing new, more complex one. The primary goal was to help parsing and transforming plain text logs into JSON objects (one line => one object) based on provided template. 

## Install
Install locally: `npm install grok-js`.

## Quick start
Following simple snippet
```javascript
const p = '%{IP:client} \\[%{TIMESTAMP_ISO8601:timestamp}\\] "%{WORD:method} %{URIHOST:site}%{URIPATHPARAM:url}" %{INT:code} %{INT:request} %{INT:response} - %{NUMBER:took} \\[%{DATA:cache}\\] "%{DATA:mtag}" "%{DATA:agent}"';
const str = '203.35.135.165 [2016-03-15T12:42:04+11:00] "GET memz.co/cloud/" 304 962 0 - 0.003 [MISS] "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36"';

require('grok-js').loadDefault((err, patterns) => {
  if (err) {
    console.error(err);
    return;
  }

  const pattern = patterns.createPattern(p);

  pattern.parse(str, (err, obj) => {
    if (err) {
      console.error(err);
      return;
    }

    console.log(obj);
  });
});
```
will transform string
```
203.35.135.165 [2016-03-15T12:42:04+11:00] "GET memz.co/cloud/" 304 962 0 - 0.003 [MISS] "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36"
```
into object
```json
{ 
  "client": "203.35.135.165",
  "timestamp": "2016-03-15T12:42:04+11:00",
  "method": "GET",
  "site": "memz.co",
  "url": "/cloud/",
  "code": "304",
  "request": "962",
  "response": "0",
  "took": "0.003",
  "cache": "MISS",
  "mtag": "-",
  "agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36" 
}
```

## Synchronous version of code
```javascript
const p = '%{IP:client} \\[%{TIMESTAMP_ISO8601:timestamp}\\] "%{WORD:method} %{URIHOST:site}%{URIPATHPARAM:url}" %{INT:code} %{INT:request} %{INT:response} - %{NUMBER:took} \\[%{DATA:cache}\\] "%{DATA:mtag}" "%{DATA:agent}"';
const str = '203.35.135.165 [2016-03-15T12:42:04+11:00] "GET memz.co/cloud/" 304 962 0 - 0.003 [MISS] "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36"';

try {
  const patterns = require('grok-js').loadDefaultSync();
  const pattern = patterns.createPattern(p);

  console.log(pattern.parseSync(str));
} catch (err) {
  console.error(err);
}
```

## Promises
Experimental
```javascript
const p = '%{IP:client} \\[%{TIMESTAMP_ISO8601:timestamp}\\] "%{WORD:method} %{URIHOST:site}%{URIPATHPARAM:url}" %{INT:code} %{INT:request} %{INT:response} - %{NUMBER:took} \\[%{DATA:cache}\\] "%{DATA:mtag}" "%{DATA:agent}"';
const str = '203.35.135.165 [2016-03-15T12:42:04+11:00] "GET memz.co/cloud/" 304 962 0 - 0.003 [MISS] "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36"';

require('grok-js').loadDefault().then(patterns => {
  return patterns.createPattern(p).parse(str);
}).then(obj => {
  console.log(obj);
}).catch(err => {
  console.log(err);
})
```

## API
* **loadDefault([loadModules,] callback)** - creates new pattern collection including all built-in patterns from `./patterns` folder. By providing *loadModules* parameter you can limit number of loaded patterns: `loadDefault(['grok-patterns'] ,...);`. Callback receives *patterns* collection filled in with default templates: `function(err, patterns)`.

* **loadDefaultSync([loadModules])** - creates new default pattern collection and returns it `GrokCollection`.

* **new GrokCollection()** - creates a new empty pattern collection.

* **GrokCollection.createPattern(expression, [id])** - creates new pattern and adds it to the collection. Find out more about pattern syntax [here](http://logstash.net/docs/1.4.2/filters/grok) and about regular expression syntax [here](http://www.geocities.jp/kosako3/oniguruma/doc/RE.txt)

* **GrokCollection.getPattern(id)** - returns existing pattern `GrokPattern`

* **GrokCollection.load(filePath, callback)** - asynchronously loads patterns from file. Callback is `function(err)`.

* **GrokCollection.loadSync(filePath)** - loads patterns from file and returns number of newly loaded patterns `number`

* **GrokPattern.parse(str, callback)** - parses string using corresponding pattern. Callback function receives optional *error* and resulting object *result*: `function(error, result)`

* **GrokPattern.parseSync(str)** - parses string using corresponding pattern and returns resulting object `object`

Find out more about grok-js https://memz.co/parsing-log-files-node-js-regex-grok/ 

## License 
**ISC License (ISC)**

Copyright (c) 2019, Andrey Chausenko <andrey.chausenko@gmail.com>

Permission to use, copy, modify, and/or distribute this software for any
purpose with or without fee is hereby granted, provided that the above
copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
