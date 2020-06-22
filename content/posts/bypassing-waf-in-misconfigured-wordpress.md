---
title: "Bypassing WAF in Misconfigured Wordpress"
date: 2020-06-22T14:45:27+05:30
draft: false
---

Web Application Firewalls like cloudflare are pretty good at protecting websites by tunneling the traffic through their secure servers. But if the underlying IP address is leaked, such protection is usually bypassed and the attacker can directly target the application.

# IP Disclosure in WordPress
WordPress stores the site url and home url in the database and uses them to serve content or redirect users. But sometimes the website is required to migrate to another domain. This can break the WordPress installation. One way to fix this is taking backup and restoring it to new domain and replacing every mention of the old domain in the database with the new domain. This is quite inconsistent and usually creates a lot of breaking if not done correctly. To overcome such inconsistency, the WordPress allows two constants ie. ```WP_HOME``` and ```WP_SITEURL``` in ```wp-config.php``` which can be used to override database entries.

If a WordPress site is using these two constants incorrectly, it can lead to old site url disclosure and in some cases, IP Address disclosure. 

## Environment
The exploitation assumes that a WordPress installation done on a server without any domain ie. has been installed directly using IP Address say ```123.456.789.012``` (This is quite common in VM hostings like AWS EC2 where an IP is allocated but the domain is not pointed yet). Further it is assumed that the developer later defined the ```WP_HOME``` constant in ```wp-config.php``` to override the old urls definitions but did not configured ```WP_SITEURL``` constant.

## Exploitation
For the target ```target.com```, the exploitation starts by following the logout route with the parameter ```redirect_to``` to any other domain:
```
https://target.com/wp-login.php?action=logout&redirect_to=https://example.com
```

The WordPress CSRF protection triggers and shows a warning message with link containing nonce token:
![](/images/22-06-2020/logout.png) 

Clicking on the logout link redirects the attacker to the url in ```redirect_to```. But due to filteration for open redirect, WordPress redirects the attacker to the value of ```admin_url()```.

```
https://target.com/wp-login.php?redirect_to=https%3A%2F%2F123.456.789.012%2Fwp-admin%2F&reauth=1
```

Since the developer did not configured ```WP_SITEURL``` the ```admin_url()``` gave the real site url from the database and in this case, real ip stored within the database.

## Internal Explanation
The logout functionality redirects the user using the function ```wp_safe_redirect``` which calls ```wp_sanitize_redirect```, ```wp_validate_redirect``` and in the end ```wp_redirect``` ie. The given argument to ```wp_safe_redirect``` is first sanitized, then validated and in the end redirected.

The function ```wp_safe_redirect``` resides in [wp-includes/pluggable.php](https://developer.wordpress.org/reference/functions/wp_safe_redirect/#source)

```php
function wp_safe_redirect( $location, $status = 302, $x_redirect_by = 'WordPress' ) {
 
    // Need to look at the URL the way it will end up in wp_redirect().
    $location = wp_sanitize_redirect( $location );
 
    /**
     * Filters the redirect fallback URL for when the provided redirect is not safe (local).
     *
     * @since 4.3.0
     *
     * @param string $fallback_url The fallback URL to use by default.
     * @param int    $status       The HTTP response status code to use.
     */
    $location = wp_validate_redirect( $location, apply_filters( 'wp_safe_redirect_fallback', admin_url(), $status ) );
 
    return wp_redirect( $location, $status, $x_redirect_by );
}
```

As you can see, the filter ```wp_safe_redirect_fallback``` uses ```admin_url()``` which falls back to the entry in the database on missing ```WP_SITEURL```.

```admin_url()``` in action:
[![asciicast](https://asciinema.org/a/aHbtzpnHZpXgRt6OexCFfsCqU.svg)](https://asciinema.org/a/aHbtzpnHZpXgRt6OexCFfsCqU)


## Mitigation
For the most WordPress installations, defining ```WP_SITEURL``` along with ```WP_HOME``` is enough to protect against such leaks. If you are for some reason don't wish to define ```WP_SITEURL```, you should change the entries within the database appropriately. On the other hand, if you are trying to migrate your website to another domain, it would be best to use a migration plugin that automagically replaces the old references.
