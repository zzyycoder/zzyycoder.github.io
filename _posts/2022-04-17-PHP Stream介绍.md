---
title: PHP Stream介绍
date: 2022-04-17
categories: []
tags: []
---
### What are streams?
Streams provide on-demand access to data. This means you don’t need to load the entire contents of your dataset into memory before processing can start. Without streams, opening a 20MB file will consume 20MB of memory.

Through streams, you can carry out read and write operations seamlessly, regardless of the context of the data. So whether your context is the file system, TCP connection, or a compressed file, you can process your data with ease.

Every stream has an implementation of a wrapper. A wrapper provides the additional code necessary to handle the specific protocols or encodings. PHP has a number of wrappers built in. It’s also easy to write wrappers of your own to interact with other services and protocols.

You reference a stream using a scheme and a target, like this: `<scheme>://<target>`. The scheme is the protocol or wrapper that’s being referenced, and the target is the specific resource identifier. The default wrapper is the file system. That means you use streams every time you interact with the file system. You’re probably already familiar with other schemes, such as HTTP and FTP.

### stream wrappers
```
>>> stream_get_wrappers()
=> [
     "https",
     "ftps",
     "compress.zlib",
     "php",
     "file",
     "glob",
     "data",
     "http",
     "ftp",
     "phar",
     "zip",
   ]
```

### The PHP wrapper
To get access to the standard input stream, you can use php://stdin, which is read-only. In contrast, php://stdout gives direct access to the standard out stream and php://stderr to the error stream, both of which are write-only.

The streams php://memory and php://temp are read-write, allowing temporary data to be stored in a file-like wrapper. The difference between the two is that php://memory will always be in memory, whereas php://temp will start writing to a temporary file when the memory limit is reached. This limit is predefined in the PHP configuration, and the default is 2MB.

### Stream Contexts
A context is a set of parameters and wrapper-specific options that can enhance or otherwise change the behavior of a stream. You create a context using the stream_context_create() function. Most of the stream creation functions will accept a context array.
```php
// You can create an array of array with your custom values.
$opts = [
  // The top-level key is the wrapper you want to alter.
  'https'=> [
    // These are keys you may want to change.
    'method'=>"GET",
    'header'=> "User-Agent: MyCoolBrowser"
  ]
];

// You can change the default by using this function and passing the array of changes.
$default = stream_context_get_default($opts);

// Now the headers will declare your User-Agent as MyCoolBrowser when you get this file.
readfile('https://www.theguardian.com');
```
People normally change headers for much more practical reasons, such as to add tracking or verification. However, you can easily extend this trivial example to each of those tasks.

### Using Filters
A filter is a final piece of code that performs operations on data as it’s being read from or written to a stream. You can stack multiple filters onto a stream.

PHP comes with built-in filters.
```
>>> stream_get_filters()
=> [
     "zlib.*",
     "string.rot13",
     "string.toupper",
     "string.tolower",
     "string.strip_tags",
     "convert.*",
     "consumed",
     "dechunk",
     "convert.iconv.*",
   ]
```
```
<?php
$emails = fopen(__DIR__ . '/big_file.txt', 'r');
stream_filter_append($emails, 'string.toupper');
```
Now when you use `fgets()`` to read the email addresses, each character will be in uppercase.
