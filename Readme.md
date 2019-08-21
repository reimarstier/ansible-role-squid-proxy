# Configure parent proxy
squid_parents:
- name: parent1
  destination: 10.8.2.1
  port: 51234
  options: round-robin

# Logging format

* https://wiki.squid-cache.org/Features/LogFormat
* http://devel.squid-cache.org/customlog/logformat.html
* https://serverfault.com/questions/359664/change-squid3-access-log
* https://wiki.squid-cache.org/Features/LogModules

# Logging format examples
* combined
```
%{%Y/%m/%d %H:%M:%S}tl %>a %un "%rm %ru HTTP/%rv" %>Hs %<st "%{Referer}>h" "%{User-Agent}>h" %Ss:%Sh

2019/05/05 10:21:26 1.2.3.4 - "CONNECT clients4.google.com:443 HTTP/1.1" 403 4071 "-"
"Chrome LINUX 73.0.3683.86 (f9b0bec6063ea50ce2b71f5b9abbae7beee319a6-refs/branch-heads/3683@{#858})
channel(unknown)" TCP_DENIED:HIER_NONE

# <Client IP> <Username> "<Request Method> <Request URL> HTTP/<Protocol Version> <Response Status Code> \
# <Sent reply size (with hdrs)> <Referer> <User Agent> <Squid Request Status>:<Squid Hierarchy Status>
```
* nourl
```
%{%Y/%m/%d %H:%M:%S}tl %6tr %>a %Ss/%03>Hs %<st %[un %Sh/%<a %mt

2019/05/05 10:14:04  10363 1.2.3.4 TCP_TUNNEL/200 524 username HIER_DIRECT/104.244.37.20 -
```

# Fail2ban issues
https://github.com/fail2ban/fail2ban/pull/1615
https://www.fail2ban.org/wiki/index.php/Fail2ban:Community_Portal#Squid_filter
https://github.com/fail2ban/fail2ban/blob/master/config/filter.d/squid.conf

* Only ban 407 authentication requests
(Source: https://wiki.squid-cache.org/Features/Authentication)

If the header is missing, Squid returns an HTTP reply with status 407 (Proxy Authentication Required).
The user agent (browser) receives the 407 reply and then attempts to locate the users credentials.
Sometimes this means a background lookup, sometimes a popup prompt for the user to enter a name and password.
The name and password are encoded, and sent in the Authorization header for subsequent requests to the proxy.

* Verify it catches only the expected lines
```
/usr/bin/fail2ban-regex /var/log/squid/access_custom.log /etc/fail2ban/filter.d/squid-nourl.local --print-all-matched
```

# SSL / YouTube
https://docs.diladele.com/faq/squid/cannot_connect_to_site_using_https.html
https://serverfault.com/questions/358754/how-to-make-squid-work-like-proxy-only-without-caching-anything
https://wiki.squid-cache.org/ConfigExamples/DynamicContent/YouTube
