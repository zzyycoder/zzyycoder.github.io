---
title: PHP and PHP-FPM
date: 2022-03-10
categories: []
tags: []
---
PHP is a source scripting language, specially designed for web application development. In 2021, PHP represented a little less than 80% of the web pages generated in the world. PHP is open-source and is the core of the most famous CMS (WordPress, Drupal, Joomla!, Magento, ...).

PHP-FPM (FastCGI Process Manager) is integrated to PHP since its version 5.3.3. The FastCGI version of php brings additional functionalities.

### Generalities

CGI (Common Gateway Interface) and FastCGI allow communication between the web server (Apache, Nginx, ...) and a development language (Php, Python, Java):
- In the case of CGI, each request leads to the creation of a new process, which is less efficient in terms of performance.
- FastCGI relies on a certain number of processes for the treatment of its client requests.

> When running PHP through your web server, there are two distinct options: running it using PHP's CGI SAPI, or running it as a module for the web server. Each have their own benefits, but, overall, the module is generally preferred.
>
> Running PHP as a CGI means that you basically tell your web server the location of the PHP executable file, and the server runs that executable, giving it the script you called, each time you visit a page. That means each time you load a page, PHP needs to read php.ini and set its settings, it needs to load all its extensions, and then it needs to start work parsing the script - there is a lot of repeated work.
>
> When you run PHP as a module, PHP literally sits inside your web server - it starts only once, loads its settings and extensions only once, and can also store information across sessions. For example, PHP accelerators rely on PHP being able to save cached data across requests, which is impossible using the CGI version.
>
> The obvious advantage of using PHP as a module is speed - you will see a big speed boost if you convert from CGI to a module. Many people, particularly Windows users, do not realise this, and carry on using the php.exe CGI SAPI, which is a shame - the module is usually three to five times faster.
>
> There is one key advantage to using the CGI version, though, and that is that PHP reads its settings every time you load a page. With PHP running as a module, any changes you make in the php.ini file do not kick in until you restart your web server, which makes the CGI version preferable if you are testing a lot of new settings and want to see instant responses.

> Since Apache has a php module, the use of php-fpm is more commonly used on an Nginx server.

### mode_php
mod_php means PHP, as an Apache module. Basically, when loading mod_php as an Apache module, it allows Apache to interpret PHP files.

Using PHP as an Apache module: the PHP interpreter is then kind of "embedded" inside the Apache process : there is no external PHP process -- which means that Apache and PHP can communicate better.

php.ini is read when the PHP module is loaded in both mod_php, FastCGI and FPM. In regular CGI mode, the config file have to be read at runtime because there's no preforked processes of any kind.
