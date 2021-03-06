# IN10 Multilanguage
> Simple, opinionated multilanguage package for Laravel

## Contents
- [Requirements](#requirements)
- [Design](#design)
- [Installation](#installation)
- [Migrating from Arcanedev/localization](#migrating-from-arcanedevlocalization)
- [Usage](#usage)
- [Developer](#developer)
- [License](#license)

## Requirements
* PHP 7.1 - 7.3
* Laravel 5.7

## Design
This is an opinionated package: it works in a specific way, based on the setups we run at IN10. That means:

1. All translated routes start with a route part, e.g. example.com/de/news/
1. A website has a pre-defined set of languages, all other languages return a 404.
1. A language is always two characters.
1. The homepage is translated.
1. The website has a single default language, by default "en" which you can change in the configuration. This default language is excluded from the URL via a 301-redirect. If you visit example.com/en/test, it will be redirected to example.com/test.

## Installation
Install the package using composer:
```bash
composer require "in10/multilanguage"
```
and publish the configuration file:
```bash
php artisan vendor:publish --provider=IN10\\Multilanguage\\ServiceProvider
```
You can customize this file as needed.

## Migrating from ARCANEDEV/localization
One of the packages we used to use at IN10 is [ARCANEDEV/Localization](https://github.com/arcanedev/localization). To facilitate an easy upgrade from the package to this new, smaller package, execute the following steps:

1. Remove the existing package: `composer remove arcanedev/localization`.
2. Remove the ServiceProvider and configuration file if needed.
3. Follow the steps in the [Installation](#installation) section above to install IN10/multilanguage.
4. Search through your project for the following instances of Localization-specific code that must be replaced

| Search for | Replace with | Remarks |
| ---------- | ------------ | ------- |
| `localization()->getCurrentLocale()` | `App::getLocale()` | Don't forget to import the Facade |
| `Localization::getCurrentLocale()` | `App::getLocale()` | Don't forget to import the Facade |
| `config('localization.supported-locales')` | `config('languages.supported-languages` | |

5. Test your project thoroughly to check if all translated routes and features still work.

## Usage

### Setting up groups
You can make a set of routes translated by wrapping them in a group:
```php
Route::multilanguage([], function() {
    Route::get('/', 'HomepageController')->name('homepage');
    Route::get('/news/{slug}', 'HomepageController')->name('news.show');
});
```
The first parameter `attributes` takes the same settings as a regular route group, except for `prefix`, `as` and `middleware`, which are overwritten (these parameters are required to make the translation work). The multilanguage-group should be a root-level construct.  Adding it inside of another group or prefix is not tested and probably won't work.

### Route translation
In some cases, you might want to translate slugs in the URL. A common example is the `/en/news/an-article` and `/nl/nieuws/een-artikel` variants of URLs. This can be accomplished using the `transGet` routing function:

```php
Route::multilanguage([], function() {
    Route::transGet('news.show');
});
```
Note that using `transGet` outside of a multilanguage routing group will not work.

The translation key is automatically looked up in the `routes.php` translation file. All translation routes must always be translated. Don't fret: the package will scream at you if you're missing a translation.
```php
// resources/lang/en/routes.php
return [
    'news.show' => 'news/{slug}',
];

// resources/lang/nl/routes.php
return [
    'news.show' => 'nieuws/{slug}',
];
```

### Route generation
If you want to generate a route with a correct language, use the included helper:
```php
function translatedRoute(string $route, array $parameters = [], bool $absolute = true, ?string $language = null) : string
```
This helpers takes the same parameters as the Laravel `route()` helper, with an optional language as a last parameter. If you omit the language, the helper uses the current language for the request. This is usually what you want, so in general you can use the translatedRoute helper as if it were the regular helper:
```php
translatedRoute('news.show', ['slug' => 'five-ways-to-translate-content');
```
which will generate `/nl/news/five-ways-to-translate-content` in this example if the current language is set to Dutch.

## Developer
[Jakob Buis](https://www.jakobbuis.nl)

## License
Copyright 2019 [IN10](https://www.in10.nl). All rights reserved.
