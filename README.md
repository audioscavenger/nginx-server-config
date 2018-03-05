# WordPress Nginx

This repository contains the Nginx configurations used within the series [Hosting WordPress Yourself](https://deliciousbrains.com/hosting-wordpress-setup-secure-virtual-server/). It contains best practices from various sources, including the [WordPress Codex](https://codex.wordpress.org/Nginx) and [H5BP](https://github.com/h5bp/server-configs-nginx). The following example sites are included:

* singlesite.com - WordPress single site install (no SSL or page caching)
* ssl.com - WordPress on HTTPS
* fastcgi-cache.com - WordPress with [FastCGI caching](https://deliciousbrains.com/hosting-wordpress-yourself-server-monitoring-caching/#page-cache)
* ssl-fastcgi-cache.com - WordPress on HTTPS with FastCGI caching
* multisite-subdomain.com - WordPress Multisite install using subdomains
* multisite-subdirectory.com - WordPress Multisite install using subdirectories

Looking for a modern hosting environment provisioned using Ansible? Check out [WordPress Ansible](https://github.com/A5hleyRich/wordpress-ansible).

## Usage

### PHP configuration

The php-fpm pool configuration is located in `global/php-pool.conf` and defaults to PHP 7.0.  It will need modified if you want the default php-fpm pool service to be a different PHP version.  Additional PHP version upstream definitions can be added to the `/upstreams` folder (a PHP 7.0 and 7.1 samples are provided there).  The default pool is defined using `$upstream` in your server configurations, or you can use specific upstream definition (i.e. php71, php70, php53, etc) if they exist under `/upstreams`.

For example, currently all the nginx vhosts have the same following set for php requests, defined under `nginx.conf` ($upstream -> php7.0 by default):

```
fastcgi_pass    $upstream
```

You could change a particular vhost server to the following to use the php 7.1 service instead (assuming that php7.1-fpm service is running).

```
fastcgi_pass    php71
```

This effectively allows you to have different server blocks execute different versions of PHP if needed.

### Installation

You can use these sample configurations as reference or directly by replacing your existing nginx directory. Follow the steps below to replace your existing nginx configuration.

Backup any existing config by moving it:

`sudo mv /etc/nginx /etc/nginx.backup`

Backup any existing config by zipping it:

`sudo tar cvf /etc/nginx /etc/nginx-$(date +"%Y%m%d").tgz`

Clone the repo:

`sudo git clone https://github.com/audioscavenger/wordpress-nginx.git /etc/nginx`

Symlink the default file from _/sites-available_ to _/sites-enabled_, which will setup a catch-all server block. This will ensure unrecognised domains return a 444 response.

Option 1: manual:
`sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/default`

Option 2: install nginx_ensite
```
git clone https://github.com/audioscavenger/nginx_ensite.git /tmp/nginx_ensite
cd /tmp/nginx_ensite
sudo make install

sudo nginx_ensite default.vhost
```

Duplicate one of the example configurations from _/sites-available_ to _/sites-available/yourdomain.com.vhost_:

`sudo cp /etc/nginx/sites-available/singlesite.com.vhost /etc/nginx/sites-available/yourdomain.com.vhost`

Edit the site accordingly, paying close attention to the server name and paths.

To enable the site, symlink the configuration into the _/sites-enabled_ directory:

Option 1: manual:
`sudo ln -s /etc/nginx/sites-available/yourdomain.com.vhost /etc/nginx/sites-enabled/yourdomain.com.vhost`

Option 2: using nginx_ensite
`sudo nginx_ensite yourdomain.com.vhost`

Test the configuration:

`sudo nginx -t`

If the configuration passes, reload Nginx:

`sudo /etc/init.d/nginx reload`

## Directory Structure

This repository has the following structure, which is based on the conventions used by a default Nginx install on Debian:

```
.
├── conf.d
├── global
    └── server
├── locations
├── sites-available
├── sites-enabled
├── upstreams
```

__conf.d__ - configurations for additional modules.

__global__ - configurations within the `http` block.

__global/server__ - configurations within the `server` block. The `defaults.conf` file should be included on the majority of sites, which contains sensible defaults for caching, file exclusions and security. Additional `.conf` files can be included as needed on a per-site basis.

__locations__ - configurations for individual locations (used under server definitions).

__sites-available__ - configurations for individual sites (virtual hosts).

__sites-enabled__ - symlinks to configurations within the `sites-available` directory. Only sites which have been symlinked are loaded.

__upstreams__ - upstreams, currently only for php-pool. One can imagine CGI and other stuff to be added here one day.

### Recommended Site Structure

The following site structure is used throughout this repository:

```
.
├── yourdomain1.com
    └── cache
    └── logs
    └── public
├── yourdomain2.com
    └── cache
    └── logs
    └── public
```

### Walkthrough default configurations provided

#### /conf.d

├── cloudflare.conf             : implement real_ip_header for Cloudflare IP forwarding (doesn't hurt if you don't use it).

├── origin-pull-ca.pem          : CloudFlare presents certificates signed by a CA with the following certificate. See https://support.cloudflare.com/hc/en-us/articles/204494148

├── cloudflare_ipv4.conf        : Cloudflare ipv4 proxies

├── cloudflare_ipv6.conf        : Cloudflare ipv6 proxies

└── webp.conf                   : webp and jxr image mappings

#### /locations

├── auth-your-site_example.conf : auth_basic and ip blocking example you can use for wp-admin location

├── grafana.conf                : grafana server proxy_pass

├── location_example.conf       : static files location example to include in a server directive

├── netdata.conf                : netdata server proxy_pass

├── prometheus.conf             : prometheus server proxy_pass

├── stub_status.conf            : nginx stub_status for monitoring, included in localhost.vhost

└── wp-admin_example.conf       : wp-admin security location to be included in a server directive

#### /global/server

Anything here is to be included under a server definition.

├── Content-Security-Policy.conf: implementation of W3C recommendations https://www.w3.org/TR/CSP1/ - included in `security.conf`

├── defaults.conf               : to be included once only. Includes `exclusions.conf`, `security-general.conf`, and `security.conf`

├── exclusions.conf             : security exclusions such as hidden files

├── fastcgi-cache.conf          : fastcgi cache buffers and timeouts; included in fastcgi vhosts

├── multisite-subdirectory.conf : WordPress multisite-subdirectory example

├── outdated.conf               : maps $outdated variable to 0|1 upon $http_user_agent value (not used in any vhost example)

├── proxy_params.conf           : to be included in servers that use proxy_pass. Included in monitoring location examples

├── security.conf               : adds multiple security headers: XSS, clickjacking and includes `Content-Security-Policy.conf`

├── security-general.conf       : unset server_tokens and limits http to GET, POST, HEAD

├── ssl.conf                    : ssl rules modernized with https://mozilla.github.io/server-side-tls/ssl-config-generator/

└── static-files.conf           : static file rules: images, scripts, etc. WebP serving enabled by default


## Todo

Decide what to enable in `fastcgi.conf`.

## Disclaimer

This is NOT a simple drop-in. You actually have to open and read each files that you are about to use for a public website. Opening a public website is sensitive and you need to understand the basic concepts of nginx. Thus, a lot of sources and comments are included for your sake.

This configuration has been tested with nginx-extra 1.10.3 on Ubuntu and may not be compatible with your version.

Do not trust me or anyone else on the web, you never know what kind of back-door you are about to install unless it's packaged by an official company. Analyse each included file carefully and do not uncomment stuff you don't have a clue about, without searching for it on the web. For instance, a lot of fastcgi configuration is currently commented because it takes me hours to read information available, then implement, then test.

If you have a question or an idea to propose, submit an issue. If are willing to participate, you are most welcome.

