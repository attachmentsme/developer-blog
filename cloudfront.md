Serving Assets Through Amazon CloudFront, For Fun and Profit
=====================================

_Can we speed up the load time of landing page to sub 2-seconds?_, this ticket had been kicking around in our backlog for quite some time. From the get go, we had made attempts to deliver assets efficiently:

* our CSS and JavaScript files were combined and minified.
* static assets were served directly through Nginx.
* Nginx was configured to gzip assets.

Regardless of these efforts, our landing page's loading times were becoming abysmal:

![Before CloudFront](./images/cloudfront/before-cf.png)
