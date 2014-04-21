Phalcon & Drupal
================
Within this reading, I will cover a setup of combining your Drupal back-end with a Phalcon front-end.

Subjects
--------
* Why? A little background information
* Requirements for setting up this work environment
* Project structure
* Drupal
    * Using JSON as cachable data layer with a simple module to hook your data
    * Using the REST service with oAuth 2.0 for more security
* Phalcon
    * Create a scalable INI configuration layer
    * Using a JSON Model to fetch data from Drupals generated JSON files
    * Using the oAuth 2.0 library with a REST Model for secure data transferring
* Optional scaling of your application
    * Multilingual support

Why? A little background information
------------------------------------
The reason I'm writing this reading is because this setup really helped me to deliver an application
that passed all given requirements. It is no secret that Drupal struggles with performance, especially
for larger websites that generate a lot of traffic and generated data.

After building our first application in Phalcon, I was very satisfied by it's fast page loads and
overall performance. Phalcon has a C extension installed as Apache module which makes it the self called
'fastest PHP Framework'.

When delivering a CMS we're always implementing Drupal and our clients are very happy with this CMS.
The 'problem' is that Drupal doesn't give us the performance Phalcon provides us with. The bootstrap is
just too slow and it's not always the best solution to implement caching layers like Varnish.

And the same way we used Flash as front-end in our past, or using mobile devices in the present. Drupal
will never fail for what it is, namely, content management. This is why  we are where we are. Using
Phalcon as our advanced MVC framework filled with content served in our client friendly Drupal CMS.

Requirements for setting up this work environment
-------------------------------------------------