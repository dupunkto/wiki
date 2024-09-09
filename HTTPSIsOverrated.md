While it's true that [[YouShouldUseHTTPS]] everywhere, in many cases, it is overkill, and unneccesarily restricts access to your websites for those running outdated browsers and unable to access newer software, due to forexample license, bandwidth or hardware limitations.

[[WebSite]]s and applications transmitting user information and handling payments should **definitely** use exclusively HTTPS to serve **all** traffic. However, for simple, text-based, non-dynamic websites that purely serve up static text and media, like most blogs and personal websites, forcing HTTPS is unneccesary.

Furthermore, HTTPS is **extremely slow**. In many cases, tests reported an overhead of up to 100%. That's _a lot_.

## So, please, disable your redirects

We're not advocating to disable HTTPS entirely, but instead we'd like to give our readers the choice, by serving our websites over both HTTP and HTTPS respectively.

This page was inspired by [1MB.club](https://1mb.club/blog/https-redirects).
