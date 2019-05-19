---
layout: post
title: "Serving Rails assets with nginx"
date: 2015-11-08 17:00
comments: false
categories: [rails, nginx, assets]
---

Using the Rails asset pipeline makes it safe to cache assets forever in browsers. Rails is configured for this out of the box, so you only need to ensure that you precompile assets before or during each deploy.

With that out of the way you can simply add this snippet to your nginx config to allow assets with digests in their name to be cached forever in clients:

{% highlight nginx %}
# From https://object.io/site/2015/rails-nginx-easy-assets
#
# Cache forever publicly: files for generated assets
#   /assets/application-2565b50fc38a0b3a44882faa3e936262.css
#
# This setup means a CDN may cache these files
location ~ "^/assets/.+-[0-9a-f]{32}.*" {
  gzip_static on;
  expires     max;
  add_header  Cache-Control public;
}

# Try serving request as a static file, no caching:
location / {
  try_files $uri/index.html $uri.html $uri;
}
{% endhighlight %}

This gives special treatment to files in `/public/assets/` with a 32-character hex digest in the filename. These files are exactly the ones built by the asset pipeline, and they are allowed to be publicly cached forever, as they are guaranteed to never change their contents. With this configuration in place you can also trivially use your nginx as the origin server for a CDN.

Any static files not matching the first rule are not allowed to be cached forever and will have to be re-requested with each visit. The more you can build as digested assets, the less strain you will see on your nginx server and your bandwidth bill.

Happy caching!
