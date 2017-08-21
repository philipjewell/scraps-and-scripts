# Scraps and Scripts
Description: Series of stuff I have collected over time that I have felt helpful - some links to gist.


## function_check.php

If you're doing shared hosting, it is extremely likely your provider has some sort of restrictions on the platform for what you can run. You can use this script to test if the function you are trying to use is restricted by that provider:

To dowload it via CLI:

```
curl -O https://raw.githubusercontent.com/philipjewell/scraps-and-scripts/master/function_check.php
```

Just swap out the `FUNCTION_HERE` in the code with the function you are trying to test and run the script. Since this is a basic if/else statement, it will allow you to see if the function is able to fun on the server. 
If you're using WP CLI, you can run the following command within the actual WordPress in case the function you're checking against is custom to one of your themes or plugins:

```
wp eval-file function_check.php --debug
```

## geo restrict nginx multiple conditions

For this example, using multiple nginx conditions we're leveraging [MaxMind's GeoIP](https://www.maxmind.com).
Here we are only allowing traffic only from the country of Brazil and the IP address of: `192.171.XXX.XXX` (our local IP address). You can find your local IP address using websites like [whatismyipaddress.com](whatismyipaddress.com) or Googling "what is my ip address" should return your local IP.
You can find the list of [country codes here](http://dev.maxmind.com/geoip/legacy/codes/iso3166/), as well as use Regex in order to pipe together IP addresses and/or countries.

Listed below are a few if statements we're placing in the nginx config of the site. This will allow us to set values and determine what the response will be depending on what those values equal:

```
set $allowed_geo 0;
set $allowed_ipaddr 0;

if ($geoip_country_code !~* BR) {
set $allowed_geo 1;
}
if ($remote_addr !~* "192.171.XXX.XXX") {
set $allowed_ipaddr 1;
}
if ("$allowed_geo:$allowed_ipaddr" = "1:1") {
return 403;
}
```

This can also be written out in this way:

```
if ($geoip_country_code !~* BR) {
set $test A;
}
if ($remote_addr !~* "192.171.XXX.XXX") {
set $test "${test}B";
}
if ($test = AB) {
return 403;
}
```

This rule allows me to see from my local machine and then was able to confirm that it was working in Brazil by using [GeoPeeker](https://geopeeker.com).


## php compatability checker with wp cli

If you have WP CLI access, it can prove to be a useful tool just like and another cli. Listed below are the following commands you can use to install, run and remove the plugin.

Download and install plugin:

```
wp plugin install php-compatibility-checker --activate
```

Run the compatibility test with desired version of PHP:

```
# PHP options are: 7.0, 5.6, 5.5, 5.4, 5.3
wp phpcompat 5.5
```

Uninstall and remove plugin:

```
wp plugin deactivate php-compatibility-checker --uninstall
```

## disable comments using wp cli

From the root of the install, you will want to run these two commands:

```
wp post list --format=ids | xargs wp post update --comment_status=closed
wp post list --format=ids | xargs wp post update --ping_status=closed
```

Source: https://gist.github.com/jplhomer/646927e569548bca4f4e

## create a directory index

Add the following to the .htaccess file for that directory or create a .htaccess file with the following:

```
Options +Indexes
```

This will allow for you to list the items in the directory and anything within it.
Example: https://philip.wpengine.com/scripts/

## number of requests per hour

This helpful little command will help you see if there has been a spike in traffic. Usually something like this will be useful when comparing the load on a server during a spike.

**Apache** access logs:

```
echo -e "\nType the sitename:"; read site; echo -e '\nhits time (hr)'; grep -oE $(date +"%Y")':[0-9]{2}' /var/log/apache2/${site}.access.log | sed -r 's/\$\(date +"%Y"\)//g' | uniq -c | column -t
```

**Apache** `error` logs:

```
echo -e "\nType the sitename:"; read site; echo -e '\nhits time (hr)'; grep -oE $(date +"%a\s%b\s%d")' [0-9]{2}' /var/log/apache2/${site}.error.log | uniq -c | column -t
```

**NGINX** access logs:

```
echo -e "\nType the sitename:"; read site; echo -e '\nhits time (hr)'; grep -oE $(date +"%Y")':[0-9]{2}' /var/log/nginx/${site}.apachestyle.log | sed -r 's/\$\(date +"%Y"\)//g' | uniq -c | column -t
```

And can even be helpful to find high traffic spikes for 504 situations,
**NGINX** `504` requests:

```
echo -e "\nType the sitename:"; read site; echo -e '\nhits time (hr)'; grep " 504 " /var/log/nginx/${site}.apachestyle.log | grep -oE $(date +"%Y")':[0-9]{2}' | sed -r 's/\$\(date +"%Y"\)//g' | uniq -c | column -t
```

## live monitor incoming PING requests

```
tcpdump ip proto \\icmp
```

## last modified WordPress plugins/themes

Helpful to find the themes and plugins modified within the last day:

```
find ./wp-content/ -maxdepth 2 -type d -mtime -1 | sed 1d | cut -d\/ -f1-4 | egrep '(plugins|themes)/'
```
You will want to run this within the root of the install, and you can modify the `-mtime` flag number in order to open the range of number of days you want to look back.

## WordPress alias home and siteurl

Using this define rule in the wp-config.php will allow you to have any domain that is mapped to the install and has its DNS pointed to us to act as the primary domain (home and siteurl) in the browser:

```
define('WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST']);
define('WP_HOME', 'http://' . $_SERVER['HTTP_HOST']);
```

Keep in mind, this can be seen as duplicate content by Google's Analytics standards.
This can be helpful when inbetween domains whether it be for marketing purposes or development.

**DO NOT USE ON MULTISITES.**
I have not seen it done before, but I only imagine it will just have all the subsites display the content of the parent site/primary install.

## WordPress relatively secure home and siteurls

Placing this in the **wp-config.php** will prevent the redirect from HTTP to HTTPS or vise versa (as long has you don't have any other force rules in nginx or apache.

There are very limited cases when something like this can be used, but it has been asked about before..

```
if ($_SERVER['HTTPS']) {
 define('WP_SITEURL', 'https://DOMAIN.COM');
 define('WP_HOME', 'https://DOMAIN.COM');
} else {
 define('WP_SITEURL', 'http://DOMAIN.COM');
 define('WP_HOME', 'http://DOMAIN.COM');
}
```

## nginx query arg redirect

This is redirecting any `ref` arguments to `newref`. Place in location block:

```
if ($args ~* ref=.*) {
rewrite ^/(.*) /$1?newref=$args_ref permanent;
}
```

## nginx block curl/by user agent

To simply block a user agent, you can use the following in nginx:

```
if ( $http_user_agent ~* "USER AGENT" ) {
return 444;
access_log off;
}
```

But if you're not wanting people to curl your site/server, but still wanting services like cron to still work (as cron often uses curl in order to work), this comes in handy:

```
if ($remote_addr !~* 127.0.0.1|SERVER.IP) {
	set $local_curl  "A"; 
}
if ($http_user_agent ~* (curl) ) {
	set $local_curl "${local_curl}B";
}
if ($local_curl = AB) {
	return 444;
	access_log off;
}
```

In the above example, you will want to swap out **SERVER.IP** address with your server's IP as well as you can define the version of curl in the `$http_user_agent` value to be a specific version if need be.
*NOTE: You can actually use this for any user agent.*

This will make it so the values aren't even logged/count against the number of visits for that site.

## nginx allow java user agent

Add in location block:

```
if ( $http_user_agent ~* Java ) {
break;
}
```

You can use this for other user agents as well that we do not allow by default. Just swap out "Java" with the desired user agent.

## disable iframe from outside servers

Placing the following will disable the iframe feature for websites outside of your current site. Place this in the a location block:

```
add_header X-Frame-Options "SAMEORIGIN";
```

## nginx make WordPress uploads only available (private) for logged in users

```
location ~* ^/wp-content/uploads/(.*) {
set $mustlogin 0;
if ( $http_cookie ~* "wordpress_logged_in_" ) {
set $mustlogin "A";
}
if ($mustlogin = 0) {
return 403;
}
proxy_pass http://localhost:6776;
}
```

## find highest number of unique ip addresses

This is a quick script, will go through and count the number of unique IP addresses per site in the Apache access logs.
This is kinda only useful when you have a feeling someone might be getting under an attack.

```
ls /var/log/apache2/*.access.log | while read logfile; do echo -n "${logfile}:"; awk '{print $1}' ${logfile} | sort -u | wc -l; done | sort -rt':' -nk2 | head
```

## recursively check php for syntax errors

```
find . -iname "*.php" -exec php -l {} \; | grep -i "Errors.parsing"
```

another rendition of the same idea if you want it more verbose:

```
find . -name \*.php -exec php -l "{}" \;
```
