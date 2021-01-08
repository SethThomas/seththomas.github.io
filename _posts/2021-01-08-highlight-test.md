---
layout: post
title:  "Syntax Highlight Test"
published: tue
---


<pre><code class="language-markup">
def foo
  puts "hello"
end
</code></pre>

```Ruby
def foo
  puts "hello"
end
```

```Javascript
const AWS = require('aws-sdk')
const s3 = new AWS.S3()

// lambda handler function
exports.handler = async function(event) {

  const buckets = s3.listBuckets().promise()
  return buckets

}
```
