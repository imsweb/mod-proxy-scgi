mod-proxy-scgi
==============

A fork of Apache's mod_proxy_scgi that handles SCRIPT_NAME and PATH_INFO sanely.


The Problem
-----------

The problem with mod_proxy_scgi out of the box is that it essentially passes the request path as `SCRIPT_NAME`, regardless of how your ProxyPass line is configured. That means applications need to know where they are "mounted", instead of being told by Apache (this is basically what `SCRIPT_NAME` exists to do). And for root-mounted applications (typial Django sites), the entire path is sent as `SCRIPT_NAME`, causing Django to think every request is for /.

See also: http://www.saddi.com/software/news/archives/78-mod_proxy_scgi,-why!!.html


Fixing It
---------

All this module does is check to see if there is a path component specified after the port in your `ProxyPass` directive, and if so, uses it (minus any trailing slash) as `SCRIPT_NAME`. It also trims the `SCRIPT_NAME` off the front of `PATH_INFO`, such that `SCRIPT_NAME` + `PATH_INFO` = request path.


Example
-------

An example configuration for a Django application mounted under `/app` may look like this:

    <VirtualHost myhost:80>
        ServerName myhost.mydomain.com
        DocumentRoot /srv/http
        <Directory /srv/http>
            Options FollowSymLinks Includes
            Require all granted
        </Directory>
        ProxySCGISendfile On
        ProxyPass /static !
        ProxyPass /app/ scgi://127.0.0.1:5555/app/
    </VirtualHost>

In the confirguation above, a request for `/app/first/second/` would be passed to Django as having `SCRIPT_NAME = /app` and `PATH_INFO = /first/second/`. That way, Django's URL handling/reversing Just Work.

If, for some reason, you want to use this fork of mod_proxy_scgi, but don't want to enable the `SCRIPT_NAME` and `PATH_INFO` handling for a certain site, you can disable this processing be setting the `proxy-scgi-stupid` environment variable:

    SetEnv proxy-scgi-stupid
