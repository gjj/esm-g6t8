# osTicket x Nginx

tl;dr, skip to the [fix](#heres-the-fix-).

## osTicket is breaking! 😯

If you're getting issues like this...

![slackchat1](https://i.imgur.com/RJXq44e.png)

Or this...

![slackchat2](https://i.imgur.com/q7rcIIl.png)

What should you do? Normally it's good to open up your inspector, and make sure you check `Log XMLHttpRequests` as this will come in handy when you debug any Ajax requests.

![inspector](https://i.imgur.com/vgbCgts.png)

But in our case, our CTO has already given us this [link](https://www.nginx.com/resources/wiki/start/topics/recipes/osticket/) on Slack, so why not take a look at that?


## Here's the fix 👨‍🔧

If you were to compare your version of the `nginx.conf` versus the one shown in the link above, you'll find that there are some slight differences. So here's what you should do.

Locate the following `location` context (it's the very last one inside your `nginx.conf`) and comment it out.

        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $osticket_path$fastcgi_script_name;
            include        fastcgi_params;
        }

Now go to the commented context immediately above and uncomment it.

        location ~ \.php$ {
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
            fastcgi_param  PATH_INFO        $path_info;
            fastcgi_pass   127.0.0.1:9000;
        }

## Why though?

Same same but different right? The reason lies in how osTicket routes its `ajax.php` API endpoint (e.g. `ajax.php/report/overview/graph?period=now`), and that's why you'd have to use the osTicket recipe posted on Nginx.

If you're curious to know what happens under-the-hood, [this article](https://www.digitalocean.com/community/tutorials/understanding-and-implementing-fastcgi-proxying-in-nginx) provides a comprehensive overview of how Nginx proxies requests over to PHP via FastCGI using the patterns specified in the `location` context and `fastcgi_param` parameters.

## Restarting Nginx gracefully

Now, the changes you made to `nginx.conf` will not be updated until you restart Nginx. What's recommended is to run `sudo service nginx reload` to [gracefully restart](https://serverfault.com/a/378585) Nginx with the new configurations. 

_P.S._ A graceful restart (the `nginx reload` command) tells the web sever to finish any active connections before restarting. This means that active visitors to your site will be able to finish downloading anything already in progress before the server restarts (source: [here](http://lifeonubuntu.com/restarting-apache-gracefully/)).

And that's it! Your osTicket tooltips and file uploads will work magically.

With ❤️,

g6t8
