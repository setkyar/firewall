# Firewall

[![Latest Stable Version](https://poser.pugx.org/pragmarx/firewall/v/stable.png)](https://packagist.org/packages/pragmarx/firewall) [![License](https://poser.pugx.org/pragmarx/firewall/license.png)](https://packagist.org/packages/pragmarx/firewall)

#### A Laravel package to help you block IP addresses from accessing your application or just some routes

### Concepts

#### Blacklist

All IP addresses in those lists will no be able to access routes filtered by the blacklist filter.

#### Whitelist

Those IP addresses can

- Access blacklisted routes even if they are in a range of blacklisted IP addresses.
- Access 'allow whitelisted' filtered routes.
- If a route is filtered by the 'allow whitelisted' filter and the IP is not whitelisted, the request will be redirected to an alternative url or route name.

### Routes

This package provides two route filters:

`'fw-block-bl'`: to block all blacklisted IP addresses to access filtered routes

`'fw-allow-wl'`: to allow all whitelisted IP addresses to access filtered routes

So, for instance, you could have a blocking group and put all your routes inside it:

```
Route::group(['before' => 'fw-block-bl'], function()
{
    Route::get('/', 'HomeController@index');
});
```

Or you could use both. In the following example the allow group will give free access to the 'coming soon' page and block or just redirect non-whitelisted IP addresses to another, while still blocking access to the blacklisted ones.

```
Route::group(['before' => 'fw-block-bl'], function()
{
    Route::get('coming/soon', function()
    {
        return "We are about to launch, please come back in a few days.";
    });

    Route::group(['before' => 'fw-allow-wl'], function()
    {
        Route::get('/', 'HomeController@index');
    });
});
```

### IPs lists

IPs (white and black) lists can be stored in array, files and database. Initially database access to lists is disabled, so, to test your Firewall configuration you can publish the config file and edit the `blacklist` or `whitelist` arrays:

```
'blacklist' => array(
    '127.0.0.1',
    '192.168.17.0/24'
    '127.0.0.1/255.255.255.255'
    '10.0.0.1-10.0.0.255'
    '172.17.*.*'
    'country:br'
    '/usr/bin/firewall/blacklisted.txt',
),
```

The file (for instance `/usr/bin/firewall/blacklisted.txt`) must contain one IP, range or file name per line, and, yes, it will search for files recursivelly, so you can have a file of files if you need:

```
127.0.0.2
10.0.0.0-10.0.0.100
/tmp/blacklist.txt
```

### Redirecting non-whitelisted IP addresses

Non-whitelisted IP addresses can be blocked or redirected. To configure redirection you'll have to publish the  `config.php` file and configure:

```
'redirect_non_whitelisted_to' => 'coming/soon',
```

### Artisan Commands

To blacklist or whitelist IP addresses, use the artisan commands:

```
  firewall:list               List all IP address, white and blacklisted.
```

##### Exclusive for database usage

```
firewall
  firewall:blacklist          Add an IP address to blacklist.
  firewall:clear              Remove all ip addresses from white and black lists.
  firewall:remove             Remove an IP address from white or black list.
  firewall:whitelist          Add an IP address to whitelist.
```

This is a result from `firewall:list`:

```
+--------------+-----------+-----------+
| IP Address   | Whitelist | Blacklist |
+--------------+-----------+-----------+
| 10.17.12.7   |           |     X     |
| 10.17.12.100 |     X     |           |
| 10.17.12.101 |     X     |           |
| 10.17.12.102 |     X     |           |
| 10.17.12.200 |           |     X     |
+--------------+-----------+-----------+
```

###Facade

You can also use the `Firewall Facade` to manage the lists:

```
$ip = '10.17.12.1';

$whitelisted = Firewall::isWhitelisted($ip);
$blacklisted = Firewall::isBlacklisted($ip);

Firewall::whitelist($ip);
Firewall::blacklist($ip, true); /// true = force in case IP is whitelisted

if (Firewall::whichList($ip))  // returns false, 'whitelist' or 'blacklist'
{
    Firewall::remove($ip);
}
```

Return a blocking access response:

```
return Firewall::blockAccess();
```

Suspicious events will be (if you wish) logged, so `tail` it:

```
php artisan tail
```

### Blocking Whole Countries

You can block a country by, instead of an ip address, pass `country:<2-letter ISO code>`. So, to block all Brazil's IP addresses, you do:

```
php artisan firewall:blacklist country:br
```

You will have to add this requirement to your `composer.json` file:

```
"geoip/geoip": "1.*"
```

You can find those codes here: [isocodes](http://www.spoonfork.org/isocodes.html)

### Installation

#### Compatible with

- Laravel 4+ and 5+

#### Installing

Require the Firewall package using [Composer](https://getcomposer.org/doc/01-basic-usage.md):

```
composer require pragmarx/firewall
```

Add the Service Provider to your app/config/app.php:

```
'PragmaRX\Firewall\Vendor\Laravel\ServiceProvider',
```

Add the Facade to your app/config/app.php:

```
'Firewall' => 'PragmaRX\Firewall\Vendor\Laravel\Facade',
```

Create the migration:

```
php artisan firewall:tables
```

Migrate it

```
php artisan migrate
```

To publish the configuration file you'll have to:

**Laravel 4**

```
artisan config:publish pragmarx/firewall
```

**Laravel 5**

```
artisan vendor:publish
```

### TODO

- Tests, tests, tests.

### Author

[Antonio Carlos Ribeiro](http://twitter.com/iantonioribeiro) 

### License

Firewall is licensed under the MIT License - see the `LICENSE` file for details

### Contributing

Pull requests and issues are more than welcome.
