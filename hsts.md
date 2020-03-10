# HTTP Strict Transport Security 🔒

## Intro

While checking [Mozilla Observatory](http://observatory.mozilla.org/), you probably should have noticed that you haven't implemented HTTP Strict Transport Security in your `nginx.conf`. But if you have, you may encounter some issues below like we did.

## But first, what is it exactly?

According to [Mozilla](https://infosec.mozilla.org/guidelines/web_security#http-strict-transport-security):

> HTTP Strict Transport Security (HSTS) is an HTTP header that notifies user agents to only connect to a given site over HTTPS, even if the scheme chosen was HTTP. Browsers that have had HSTS set for a given site will transparently upgrade all requests to HTTPS. HSTS also tells the browser to treat TLS and certificate-related errors more strictly by disabling the ability for users to bypass the error page.

Too abstract? Here's an example in our context that may make the issue clearer to you. Assuming you have HSTS implemented in your Nginx configuration:
1. When you visit `http://g9t9esm2.team:8000` for the first time, you can.
2. Next, go to `http://g9t9esm2.team` and you should be redirected to `https://g9t9esm2.team`, and you'll be given the `Strict-Transport-Security` headers which will be cached by your browser.
3. Now, revisit step 1 and visit `http://g9t9esm2.team:8000` again, you'll be redirected to `https://g9t9esm2.team:8000` (note the `s` in `https`) and your browser will probably report an SSL protocol error.

Some of you may think that your CAT system is down, but it's not! That's because with the HSTS headers, your browser will always visit the HTTPS version even if there isn't one (see step 3) - and hence, it reports the SSL protocol error.

At this point, if you'd like to visit your unsecured CAT system, you probably have to clear your browser cache or visit `http://g9t9esm2.team:8000` in incognito mode.

This is intended behaviour, according to [this post](https://serverfault.com/a/882331/327653) which cites [IETF RFC 6797](https://tools.ietf.org/html/rfc6797#section-8.3) (an Internet standards document that describes how HSTS should work).

     The UA MUST replace the URI scheme with "https" [RFC2818], and

     if the URI contains an explicit port component of "80", then
     the UA MUST convert the port component to be "443", or

     if the URI contains an explicit port component that is not
     equal to "80", the port component value MUST be preserved;
     otherwise,

     if the URI does not contain an explicit port component, the UA
     MUST NOT add one.

     NOTE:  These steps ensure that the HSTS Policy applies to HTTP
            over any TCP port of an HSTS Host.


## Possible solution

One way you can fix this is to upgrade CAT to be served over HTTPS. You need to use the same SSL certificate and private key generated using the guides posted on Slack. Here's what we did to [force HTTPS redirect for non-standard ports](https://ma.ttias.be/force-redirect-http-https-custom-port-nginx/).

    server {
          listen       8000 ssl http2 default_server;
          listen       [::]:8000 ssl http2 default_server;
          
          server_name  localhost;
          include /etc/nginx/default.d/*.conf;
          
          ssl_certificate     "/home/ec2-user/ssl/certificate.crt"; # You can put this anywhere, just make sure to change this
          ssl_certificate_key "/home/ec2-user/ssl/private.key"; # and this
          
          error_page    497 https://$host:8000$request_uri;
          
          # Removed for brevity
    }

But there's a caveat if you're using AWS Application Load Balancer: you cannot use this solution. Even if you did, you'll have trouble configuring the listeners. The reason is because AWS ALB does not allow you to listen to both HTTP and HTTPS using the same port number 8000 at the same time.

Consider using a subdomain in this case, and maybe get approval from CTO while doing so.

With ❤️,

g6t8
