---
title: zend opcache的最佳设置
date: 2022-04-13
categories: []
tags: []
---
`opcache.revalidate_freq` - Basically put, how often (in seconds) should the code cache expire and check if your code has changed. 0 means it checks your PHP code every single request (which adds lots of stat syscalls). Set it to 0 in your development environment. Production doesn't matter because of the next setting.

`opcache.validate_timestamps` - When this is enabled, PHP will check the file timestamp per your opcache.revalidate_freq value.

When it's disabled, opcache.revaliate_freq is ignored and PHP files are NEVER checked for updated code. So, if you modify your code, the changes won't actually run until you restart or reload PHP (you force a reload with kill -SIGUSR2). Yes, this is a pain in the ass, but you should use it. Why? While you're updating or deplying code, new code files can get mixed with old ones— the results are unknown. It's unsafe as hell.

`opcache.max_accelerated_files` - Controls how many PHP files, at most, can be held in memory at once. It's important that your project has LESS FILES than whatever you set this at. My codebase has ~6000 files, so I use the prime number 7963 for maxacceleratedfiles.
You can run "find . -type f -print | grep php | wc -l" to quickly calculate the number of files in your codebase.

`opcache.memory_consumption` - The default is 64MB, I set it to 192MB because I have a ton of code. You can use the function opcachegetstatus() to tell how much memory opcache is consuming and if you need to increase the amount (more on this next week).

`opcache.interned_strings_buffer` - A pretty neat setting with like 0 documentation. PHP uses a technique called string interning to improve performance— so, for example, if you have the string "foobar" 1000 times in your code, internally PHP will store 1 immutable variable for this string and just use a pointer to it for the other 999 times you use it. Cool. This setting takes it to the next level— instead of having a pool of these immutable string for each SINGLE php-fpm process, this setting shares it across ALL of your php-fpm processes. It saves memory and improves performance, especially in big applications.

The value is set in megabytes, so set it to "16" for 16MB. The default is low, 4MB.

`opcache.fast_shutdown` - Another interesting setting with no useful documentation. "Allows for faster shutdown". Oh okay. Like that helps me. What this actually does is provide a faster mechanism for calling the deconstructors in your code at the end of a single request to speed up the response and recycle php workers so they're ready for the next incoming request faster. Set it to 1 and turn it on.
> This directive has been removed in PHP 7.2.0. A variant of the fast shutdown sequence has been integrated into PHP and will be automatically used if possible.

in php.ini
```ini
opcache.revalidate_freq=0
opcache.validate_timestamps=0 (comment this out in your dev environment)
opcache.max_accelerated_files=7963
opcache.memory_consumption=192
opcache.interned_strings_buffer=16
opcache.fast_shutdown=1
```
