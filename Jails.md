In FreeBSD, jails are containerised, sandboxed environments for running software, comparable with chroots on steriods.

## Setup

Here's how to set that up (in my case in `~/services`). Start by downloading the base system to your destination directory:

		mkdir ~/services
		curl https://download.freebsd.org/ftp/releases/amd64/amd64/13.2-RELEASE/base.txz -o ~/services/base.txz

Then write the jails configuration to `/etc/jail.conf`:

		exec.start = "/bin/sh /etc/rc";
		exec.stop = "/bin/sh /etc/rc.shutdown";
		exec.clean; # clear previous env
		exec.consolelog = "/var/log/jail_console_${name}.log";
		allow.nomount;
		allow.raw_sockets;
		mount.devfs;
		host.hostname = "${name}";
		path = "/home/axcelott/services/${name}";
		# Inherit network from host
		ip4 = inherit;
		ip6 = inherit;
		# Add jails here
		miniflux {
		  allow.sysvipc = true; # Make postgres happy
		}
		radicale {}

Then enable the jails:

		doas sysrc jail_enable="YES"
		doas sysrc jail_parallel_start="YES"

And finally create your jail (replacing 'miniflux' with the name of your jail):

		mkdir ~/services/miniflux
		tar -xf ~/services/base.txz -C ~/services/miniflux --unlink
		cp /etc/resolv.conf ~/services/miniflux/etc/resolv.conf
		cp /etc/localtime ~/services/miniflux/etc/localtime
		doas chown -R root:wheel ~/services/miniflux
		doas chmod 777 ~/services/miniflux/tmp
		doas freebsd-update -b ~/services/miniflux fetch install
		doas service jail start miniflux

### Setup Miniflux

Let's look at setting up Miniflux. Start by installing the necessary packages in the jail, by running the following command *on the host*:

		pkg -j miniflux install miniflux postgresql15-server postgresql15-client postgresql15-contrib

After installing the dependencies, you can enter the jail (`doas jexec miniflux /bin/sh`) to finish setting everything up:

		sysrc postgresql_enable="YES"
		sysrc miniflux_enable="YES"
		/usr/local/etc/rc.d/postgresql initdb
		service postgresql start
		sockstat -46 | grep 5432
		passwd postgres
		su - postgres
		createuser -P miniflux
		createdb -O miniflux miniflux2
		psql miniflux2 -c 'create extension hstore'
		exit
		miniflux -migrate
		miniflux -create-admin
		service miniflux start

And, finally, setup a reverse proxy using for example Nginx (in `/usr/local/etc/nginx/nginx.conf`)--on the host of course:

		server {
			listen 80;
			listen [::]:80;
			server_name reader.geheimesite.nl;

			location / {
				proxy_pass http://localhost:8080;
				proxy_set_header Host $host;
				proxy_set_header X-Real-IP $remote_addr;
				proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
				proxy_set_header X-Forwarded-Proto $scheme;
			}
		}

And (optionally), request a Let's Encrypt certificate using certbot:

		certbot --nginx -d reader.geheimesite.nl 

### Setup Radicale

After creating the jail, begin by installing dependencies from the host:

		pkg -j radicale install www/radicale apache24

(Apache is needed for htpasswd)

After that, enter the jail (`doas jexec radicale /bin/sh`) and execute the following commands to setup the radicale server:

		sysrc radicale_enable=YES
		htpasswd -Bc /usr/local/etc/radicale/users axcelott
		chown root:radicale /usr/local/etc/radicale/users
		chmod 640 /usr/local/etc/radicale/users
		mkdir -p /var/radicale/collections
		chown radicale:radicale /var/radicale

Now create a configuration file for Radicale in `/usr/local/etc/radicale/config`:

		[server]
		hosts = localhost:5232
		[auth]
		type = htpasswd
		htpasswd_filename = /usr/local/etc/radicale/users
		htpasswd_encryption = bcrypt
		[rights]
		type = owner_only
		[storage]
		filesystem_folder = /usr/local/share/radicale/collections

And finally, actually start it:

		service radicale start

On the host, we once again setup a reverse proxy using Nginx:

		server {
			listen 80;
			listen [::]:80;
			server_name contacts.geheimesite.nl;
		
			location / {
				proxy_pass http://localhost:5232;
				proxy_set_header Host $host;
				proxy_set_header X-Real-IP $remote_addr;
				proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
				proxy_set_header X-Forwarded-Proto $scheme;
			}
		}

And request the SSL certs:

		certbot --nginx -d contacts.geheimesite.nl

That's it! You can now add as many users as you like using the following command:

		doas jexec radicale htpasswd -B /usr/local/etc/radicale/users USERNAME
