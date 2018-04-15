[![Author](https://img.shields.io/badge/author-Daniel%20M.%20Hendricks-lightgrey.svg?colorB=9900cc )](https://www.danhendricks.com)
[![Latest Release](https://img.shields.io/github/release/dmhendricks/wordpress-network-subdomain-updater-plugin.svg)](https://github.com/dmhendricks/wordpress-network-subdomain-updater-plugin/releases)
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://paypal.me/danielhendricks)
[![License](https://img.shields.io/badge/license-GPLv2-yellow.svg)](https://raw.githubusercontent.com/dmhendricks/wordpress-network-subdomain-updater-plugin/master/LICENSE)
[![CloudVerve, LLC](https://img.shields.io/badge/style-FlyWheel-green.svg?style=flat&label=get%20hosted&colorB=AE2A21)](https://getflywheel.com/why-flywheel/?utm_source=github.com&utm_medium=campaign&utm_content=button&utm_campaign=wordpress-network-subdomain-updater-plugin)
[![Twitter](https://img.shields.io/twitter/url/https/github.com/dmhendricks/wordpress-network-subdomain-updater-plugin.svg?style=social)](https://twitter.com/danielhendricks)

# Network Sub-domain Updater for WordPress

A simple MU plugin that updates the primary domain on WordPress Network installations. It was primarily created for development and staging environments to make it easier when you sync data from a production site.

**Use are your own risk. This plugin is useless to you if you are *not* running a [WordPress Network](https://codex.wordpress.org/Create_A_Network) in [Subdomain mode](https://codex.wordpress.org/Create_A_Network#Installing_a_Network)!**

### How It Works

Let's say you run a WordPress Network of sub-domains. If you did a MySQL dump from your production server (example.com) and then imported to your staging (staging.example.com) or local development (example.local) instance, the primary domain would no longer match the environment.

This plugin checks to see if the primary domain changed, and if so, updates the sub-sites and relevant database fields.

For example, I use [Local by Flywheel](https://local.getflywheel.com/) for local WordPress development:

1. Create a local WordPress multisite instance.
2. Copy necessary file assets (plugins, themes, uploads, etc) from the remote site.
3. Add this mu-plugin to your local instance's `mu-plugins` directory.
4. Define the appropriate [constants](https://github.com/dmhendricks/wordpress-network-subdomain-updater-plugin#configuration) in `wp-config.php`.

Now, each time that you import MySQL dumps from your remote server to your local development database, this plugin will update the primary network and sub-sites to your local domain.

#### Important Note

This plugin will only update the domains in `wp_site`, `wp_blogs`, `wp_options`, etc to that your web site will load properly. It **will not** do a global find-and-replace for other/hard-coded domains. If you wish to do that **after** running this plugin, I recommend [Better Search Replace](https://wordpress.org/plugins/better-search-replace/).

## Installation

**NB!** This plugin is only intended to be used in staging and development environments! It is not intended to be run on production servers as it will incur a performance hit. You **can** use it to rename the primary domain on a production instance (create a MySQL backup first!), but once it's run and the domains are renamed, you should remove the plugin (or at least disable it with `NETWORK_LOCAL_DOMAIN_DISABLE`).

To install, simply download the `network-subdomain-updater.php` file and place it in your `wp-content/mu-plugins` directory (create one if it doesn't exist). Don't copy the other files - they are not necessary.

## Configuration

### Constants

This plugin is configured with constants that you defined in your development/staging environments `wp-config.php` files. **Why constants and not settings in WP Admin?** Because they would get blown away every time you import a fresh database dump.

**SITE_ID_CURRENT_SITE** *(required)*

```php
define( 'SITE_ID_CURRENT_SITE', 1 );
```

**NOBLOGREDIRECT** *(required)*

```php
define( 'NOBLOGREDIRECT', true ); // Must be set to true!
```

**NETWORK_LOCAL_DOMAIN_DISABLE**

```php
define( 'NETWORK_LOCAL_DOMAIN_DISABLE', true );
```

Allows you to temporarily disable the plugin from loading at all, useful if you do not sync your data frequently.

**NETWORK_LOCAL_DOMAIN_SCHEME**

Examples:
```php
define( 'NETWORK_LOCAL_DOMAIN_SCHEME', 'https' ); // Prefix site URLs with: https
define( 'NETWORK_LOCAL_DOMAIN_SCHEME', 'http' ); // Prefix site URLs with: http
```

(This allows you to force HTTP or HTTPS on updated site URLs, depending on what your LAMP environment supports. If not defined or set to `false`, original URL prefixes will be preserved.)

**NETWORK_LOCAL_DOMAIN_STRIP_WWW**

Examples:
```php
define( 'NETWORK_LOCAL_DOMAIN_STRIP_WWW', true );
```

Strips `www.` from domains, useful if your local development environment doesn't alias it.

**WP_ADMIN_EMAIL**

If specified, changes the `admin_mail` value in `wp_options`, useful for debugging:
```php
define( 'WP_ADMIN_EMAIL', 'you@example.com' );
```

(This is irrelevant if you are using something like [MailHog](https://github.com/mailhog/MailHog) to intercept e-mails.)

**NETWORK_UPDATE_SUBSITE_ADMIN_EMAIL**

If set to true, plugin will update the `admin_email` for all sub-sites to `WP_ADMIN_EMAIL` (rather than only the network admin e-mail):
```php
define( 'NETWORK_UPDATE_SUBSITE_ADMIN_EMAIL', true );
```

Used in conjunction with `WP_ADMIN_EMAIL`, which must also be set.

**NETWORK_LOCAL_UPDATE_NOTIFY**

If defined, sends a notification e-mail stating that an update was performed.

Send to `admin_email` using plugin defaults:
```php
define( 'NETWORK_LOCAL_UPDATE_NOTIFY', true );
```

Send to specific e-mail address:
```php
define( 'NETWORK_LOCAL_UPDATE_NOTIFY', 'you@example.com' );
```

PHP 7 or higher Options:
```php
define( 'NETWORK_LOCAL_UPDATE_NOTIFY', [ 'email' => 'you@yourdomain.com', 'subject' => 'Site sync!', 'message' => 'This space intentionally left blank.' ] );
```

### Use Case

Let's assume the following:

- Your production sites are at: `*.example.com`
- Your staging environment sites are at: `*.staging.example.com`
- Your development environment sites are at: `*.example.local`

In your *staging* environment's `wp-config.php`, you might define `DOMAIN_CURRENT_SITE` as follows:
```php
define( 'DOMAIN_CURRENT_SITE', 'staging.example.com' );
```

In your *development* environment's `wp-config.php`, you might define `DOMAIN_CURRENT_SITE` as follows:
```php
define( 'DOMAIN_CURRENT_SITE', 'example.local' );
```

### Typical Example

Below is a typical example of the constants that you'd add/change in `wp-config.php`:

```php
// Defined for multisite
define( 'DOMAIN_CURRENT_SITE', 'mysite.local' );
// ...

// Constants specific to this plugin
define( 'WP_ADMIN_EMAIL', 'webmaster@mysite.local' ); // Optional
define( 'NETWORK_LOCAL_DOMAIN_STRIP_WWW', true );
define( 'NETWORK_LOCAL_DOMAIN_SCHEME', 'https' ); // Optional
define( 'NOBLOGREDIRECT', true ); // Required
```

### Last Sync Date

This plugin will store the last sync date in a network settings field named `_nsu_last_data_sync`. In development and staging, I display this date in the [WP Admin footer](https://web-design-weekly.com/snippets/modify-footer-text-in-wordpress-admin-area/), but you may use as you wish.

```php
echo get_site_option( '_nsu_last_data_sync', 'unknown' );
```

## Change Log

Release changes are noted on the [Releases](https://github.com/dmhendricks/wordpress-network-subdomain-updater-plugin/releases) page.

#### Branch: `master`

* None since release
