# Installing Nginx & Running it as a Service

***Note 1:** There is an issue running NSSM created services on the Windows 10 Creator's Update, after you've created the services run the following command for each service `nssm set ServiceName AppNoConsole 1`  in cmd (run as admin)

***Note 2:** For the above to work, you would also need to grab the latest pre-release build from https://nssm.cc/builds
## Step 1: Installing Nginx



1. Download Nginx from: http://nginx.org/en/download.html
2. Install Nginx to your preferred location
3. Copy the Nginx folder to your preferred location e.g. c:\
4. If you want to run Nginx as a service then go the 'Running Nginx as a service' section, if not continue.
5. Go to the location you copied the Nginx folder e.g. c:\nginx
6. Double click on 'nginx.exe' in c:\nginx , nginx should now be running on your system
7. To verify, open a browser and type localhost and press enter. If you get "Welcome to nginx!” message then Nginx has been installed successfully. <br> 
**Note:** you would need to open 'nginx.exe' every time you reboot your system, to avoid this, install Nginx as a service.

## Step 2: Running Nginx as a service
1. Download NSSM from: https://nssm.cc/download
2. Copy the nssm.exe from the win32 or win64 folder depending on your system to C:\Windows\System32
3. Open cmd as admin, navigate to C:\Windows\System32
4. Type in this command without the quotes “nssm install nginx”
5. Path = C:\nginx\nginx.exe
6. Startup directory = C:\nginx

<a href="http://imgur.com/h21f0uB"><img src="http://i.imgur.com/h21f0uB.png" title="source: imgur.com" /></a>

7. Install service
8. Make sure you run the service as the admin account
 * Open run and type in services.msc
 * Search for the nginx service we just installed
 * Double-click and go to the Log On tab
 * Select ‘This account:’ and fill in your account details and then press ok.
 * Right click on the nginx service and restart
	
9. Making your Nginx install PHP ready, uncomment the following code from your nginx.conf file (c:\nginx\conf\nginx.conf)

```
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            include        fastcgi_params;
        }
```
OR

```
        #pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  c:/nginx/html$fastcgi_script_name;
            include        fastcgi_params;
        }
```




10. To verify, open a browser and type localhost and press enter. If you get "Welcome to nginx!” message then Nginx has been installed successfully.

# Installing PHP & Running as a service on Windows
***Note 1:** There is an issue running NSSM created services on the Windows 10 Creator's Update, after you've created the services run the following command for each service `nssm set ServiceName AppNoConsole 1`  in cmd (run as admin)

***Note 2:** For the above to work, you would also need to grab the latest pre-release build from https://nssm.cc/builds

## Installing PHP for Windows

1. Download PHP for Windows from here: http://windows.php.net/download (Non Thread Safe version used in this guide)
2. Create a folder called PHP under your Nginx directory e.g. C:\nginx\php and copy the downloaded files to this folder

## Running PHP as a service
Install NSSM by following steps 1 & 2 from [here.](https://github.com/causefx/Organizr/wiki/Windows-Installation-from-scratch#step-2-running-nginx-as-a-service)
1. If you’ve got ‘nssm’ already setup, open command prompt as admin. 
2. Type in the following cmd without the quotes “nssm install php”
3. Path = C:\nginx\php\php-cgi.exe
4. Startup directory = C:\nginx\php
5. Arguments =     -b 127.0.0.1:9000
6. Install Service

<a href="http://imgur.com/KTKCbDs"><img src="http://i.imgur.com/KTKCbDs.png" title="source: imgur.com" /></a>

7. On the opened cmd prompt type in “nssm start php” (without the quotes) to start the PHP service.
8. If the installed PHP service doesn’t start, then try manually running the ‘php-cgi.exe’ file in C:\nginx\php\  , If you get a missing ‘VCRUNTIME’ related error then follow the solution on this link: http://stackoverflow.com/questions/30811668/php-7-missing-vcruntime140-dll
9. Make a copy  of one of the php.ini-development or php.ini-production files and rename it to php.ini
10. Open the php.ini file and search for the following and uncomment each:
* extension_dir = "ext"
* extension=php_openssl.dll
* extension=php_pdo_sqlite.dll
* extension=php_curl.dll
11. On the opened cmd prompt type in “nssm restart php” (without the quotes) to restart the PHP service to apply the changes in php.ini.


**Note:** Organizr requires php_pdo_sqlite.dll & php_openssl.dll extensions

# Sample Config Files

**Nginx.conf**

You can copy the following if you wish and replace the content in your nginx.conf file

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.php index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        #pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  c:/nginx/html$fastcgi_script_name;
            include        fastcgi_params;
        }

         #deny access to .htaccess files, if Apache's document root
         #concurs with nginx's one
        
        location ~ /\.ht {
            deny  all;
        }
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}



```
