# PhealNG
[![Latest Stable Version](https://poser.pugx.org/3rdpartyeve/phealng/v/stable.png)](https://packagist.org/packages/3rdpartyeve/phealng)
[![Total Downloads](https://poser.pugx.org/3rdpartyeve/phealng/downloads.png)](https://packagist.org/packages/3rdpartyeve/phealng)

Copyright (C) 2012, 2013 by Peter Petermann
All rights reserved.

PhealNG is a refactoring of Pheal to meet more modern PHP development standards, for example to support namespaces
and PSR-0 classloading.

## LICENSE
Pheal is licensed under a MIT style license, see LICENSE.txt
for further information

## FEATURES
- does not need to change when EVE API changes

## REQUIREMENTS
- PHP 5.4+

## INSTALLATION

### composer
PhealNG will be available as package 3rdpartyeve/phealng through packagist on composer http://getcomposer.org, if you
don't know composer yet, now is a good time to read up on it.

You can also download it from github, but life is so much easier with composer!

## PhealNG Usage

### Prequesites

#### composer.json
In your project root create a file named composer.json, containing
the following json data:

```json
{
    "require": {
        "3rdpartyeve/phealng": "0.2.7"
    }
}
```
_Note:_ replace 0.2.6 by what ever version you want to use, you can use dev-master
but it will likely be unstable.

_Hint:_ [![Latest Stable Version](https://poser.pugx.org/3rdpartyeve/phealng/v/stable.png)](https://packagist.org/packages/3rdpartyeve/phealng)
#### composer
Composer is a tool intended to manage dependencies in PHP applications.
If you haven't installed composer yet, checkout the composer installation here:
http://getcomposer.org/doc/00-intro.md#installation-nix

#### run composer
Now in your projects root, run the following command, which will
download PhealNG for you and update the vendor/autoload.php
```bash
$ php composer.phar install
```

### the most basic PhealNG Script
this is a very basic example, which should be able to run on its own
(with only pheal as dependency) obviously, if you use a framework, some
of the steps might look different, for example if you use symfony2,
you might already have composer working and already have the autoloading
required.

PhealNG basically builds the request by the scope and the method called,
in this example the scope used is "server" and the API page requested is
ServerStatus.xml.aspx

For more information on the API Pages available, refer to
http://wiki.eve-id.net/APIv2_Page_Index

```php
<?php
require_once 'vendor/autoload.php';

//import namespace
use Pheal\Pheal;

// create pheal object with default values
// so far this will not use caching, and since no key is supplied
// only allow access to the public parts of the EVE API
$pheal = new Pheal();

// requests /server/ServerStatus.xml.aspx
$response = $pheal->serverScope->ServerStatus();

echo sprintf(
  "Hello Visitor! The EVE Online Server is: %s!, current amount of online players: %s",
  $response->serverOpen ? "open" : "closed",
  $response->onlinePlayers
);
```

### a bit more complex script
most API Pages require an API key with a certain set of rights available,
in this example we will request char/CharacterSheet.xml.aspx, which requires
a characterID passed as parameter, aswell as an API Key that contains the
AccessMask 8.


```php
<?php
require_once 'vendor/autoload.php';

//import namespace
use Pheal\Pheal;
use Pheal\Core\Config;

// the information required by this example, usually, your application would
// prompt your user for this, and/or use its database to read those information
// information like the characterID can be obtained through the EVE API,
// please check the documentation at http://wiki.eve-id.net/APIv2_Page_Index for more information
$keyID = 123456;
$vCode = "AbcDEFghYXZadfADFasdFASDFasdfQWERGHADAQerqEFADSFASDfqQER";
$characterID = 1234567;

// Pheal configuration
// Pheal may be configured through variables at the \Pheal\Cache\FileStorage Singleton object
// this allows to use different fetchers, caches, archives etc.

// setup file cache - CCP wants you to respect their cache timers, meaning
// some of the API Pages will return the same data for a specific while, or worse
// an error. If you use one of the availabe caching implementations,
// pheal will do the caching transparently for you.
// in this example we use the file cache, and configure it so it will write the cache files
// to /tmp/phealcache
Config::getInstance()->cache = new \Pheal\Cache\FileStorage('/tmp/phealcache/');

// The EVE API blocks applications which cause too many errors. Requesting a page
// that the API key does not allow to request is one of those possible errors.
// Pheal can be configured so pheal will request the AccessMask of a specific key
// and block requests to API Pages not covered by that key.
Config::getInstance()->access = new \Pheal\Access\StaticCheck();

// create pheal object with default values
// so far this will not use caching, and since no key is supplied
// only allow access to the public parts of the EVE API
//
// in this example, instead of using the scopenameScope getter,
// we set the scope directly in the constructor
$pheal = new Pheal($keyID, $vCode, "char");

try {
    // parameters for the request, like a characterID can be added
    // by handing the method an array of those parameters as argument
    $response = $pheal->CharacterSheet(array("characterID" => $characterID));

    echo sprintf(
        "Hello Visitor, Character %s was created at %s is of the %s race and belongs to the corporation %s",
        $response->name,
        $response->DoB,
        $response->race,
        $response->corporationName
    );

// there is a variety of things that can go wrong, like the EVE API not responding,
// the key being invalid, the key not having the rights to call the method
// or the characterID beeing wrong - just to name a few. So it is basically
// a good idea to catch Exceptions. Usually you would want to log that the
// exception happend and then decide how to inform the user about it.
// In this example we simply catch all PhealExceptions and display their message
} catch (\Pheal\Exceptions\PhealException $e) {
    echo sprintf(
        "an exception was caught! Type: %s Message: %s",
        get_class($e),
        $e->getMessage()
    );
}
```

### Configuration Options
In the previous example there is already two configuration options introduced,
however there are quite a few more. For more information it is worth reading the
contents of the vendor/3rdpartyeve/phealng/lib/Pheal/Core/Config.php file.


## Problems / Bugs
if you find any problems with PhealNG, please use githubs issue tracker at https://github.com/3rdpartyeve/phealng/issues

## TODO
- more documentation

## LINKS
- [Github](http://github.com/3rdpartyeve/phealng)
- [devedge](http://devedge.eu/project/pheal/)

## CONTACT
- Peter Petermann <ppetermann80@googlemail.com>

## Contributors
- Daniel Hoffend (Wollari)

## ACKNOWLEDGEMENTS
- PhealNG is based on [Pheal](http://github.com/ppetermann/pheal)
- PhealNG is written in [PHP](http://php.net)
- Pheal is based on [EAAL](http://github.com/3rdpartyeve/eaal)
- Pheal is build for use of the [EVE Online](http://eveonline.com) API
