
```bash
# https://hostnextra.com/learn/paths/caddy/install-caddy-on-ubuntu
# Ubuntu 22.04

sudo apt update

sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https

curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg

curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list

sudo apt update
sudo apt install caddy -y


systemctl status caddy

- Logs: `journalctl --no-pager -u caddy`
- ls -la /var/www/html #Are your site's files readable by the caddy user and group?
- - Is the caddy home directory writeable? `ls -la /var/lib/caddy`


mkdir -p /var/www/html

cat >/var/www/html/index.html
<!DOCTYPE html>
<html>
<head>
<title>Hello from Caddy!</title>
</head>
<body>
<h1>Hello World</h1>
</body>
</html>


vim /etc/caddy/Caddyfile
# The Caddyfile is an easy way to configure your Caddy web server.
#
# Unless the file starts with a global options block, the first
# uncommented line is always the address of your site.
#
# To use your own domain name (with automatic HTTPS), first make
# sure your domain's A/AAAA DNS records are properly pointed to
# this machine's public IP, then replace ":80" below with your
# domain name.

:80 {
        # Set this path to your site's directory.
        root * /usr/share/caddy

        # Enable the static file server.
        file_server

        # Another common task is to set up a reverse proxy:
        # reverse_proxy localhost:8080

        # Or serve a PHP site through php-fpm:
        # php_fastcgi localhost:9000
}




http://13.126.121.104 {

	root * /var/www/html
	file_server
	log {
        output file /var/log/caddy/access.log
        format console
	}
}


www.cloudforge.cloud {

	redir https://cloudforge.cloud{uri}
}

cloudforge.cloud {

	root * /var/www/html
	file_server
	log {
        output file /var/log/caddy/access.log
        format console
	}
}


systemctl restart caddy
systemctl reload caddy

# https://www.rosehosting.com/blog/how-to-install-caddy-web-server-on-ubuntu-22-04/
# https://lightsail.aws.amazon.com/ls/webapp/home/instances?#

www.cloudforge.cloud {

	redir https://cloudforge.cloud{uri}
}
cloudforge.cloud {

	reverse_proxy localhost:5000
}

systemctl restart caddy
systemctl reload caddy

```

[[Sevice5000]]
