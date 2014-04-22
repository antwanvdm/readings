Phalcon & Drupal
================
Within this reading, I will cover a setup of combining your Drupal back-end with a Phalcon front-end.
The subjects will cover the whole process so you will be ready to start your own project with this setup!

Subjects
--------
* [Why? A little background information](#why-a-little-background-information)
* [Requirements for setting up this work environment](#requirements-for-setting-up-this-work-environment)
* [Project structure](#project-structure)
* [Drupal](#drupal)
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
Anyway, we are ready and set to set up our environment so we can create some awesome code.

For our Drupal setup we need a Drupal 7 release (used 7.26 for this example, so I would advice this
version or higher) with the following contrib modules:

* Ctools (Always needed for most other contrib modules)
* Services / REST Server (Needed for Services layer with REST endpoint)
* oAuth Authentication / oAuth / oAuth provider UI (used for a safe REST Service)
* Libraries (Required by REST Server)

For a normal setup it will probably happen that you will use views to have more data control, but it's
not necessary for out basic setup.

For Phalcon I've used Phalcon 1.2.6. I don't know if the newer 1.3.x release will break anything but you
can always play with the Apache module to change the version while developing.

As Libraries you need a Httpful folder with the very handy httful rest library (http://phphttpclient.com/) &
the oAuth client (TODO: add custom link for PHP 5.3 Phalconized oAuth lib.)

Project structure
-----------------
Anyway, we are ready and set to set up our environment so we can create some awesome code. The project
setup I used was divided in the following folder structure:

* web (filled with Phalcon code)
    * public (JS/CSS/Images & main index.php file)
    * app (models/views/controllers/config/libraries/classes)
* cms (Filled with the contents of your favorite Drupal distribution/installation)
* .htaccess with following contents:

```
<IfModule mod_rewrite.c>
    RewriteEngine on

    #Always web, with the exception of Drupal
    RewriteCond %{REQUEST_URI} !/cms
    RewriteRule  (.*) web/public/$1 [L]
</IfModule>
```

After setting up this environment we should be set to start adding some custom code to our project so we
can actually connect our 2 platforms!

Drupal
------
