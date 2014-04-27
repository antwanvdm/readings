Phalcon & Drupal
================
Within this reading, I will cover a setup of combining your Drupal back-end with a Phalcon front-end.
The subjects will cover the process of providing & reading data so you will be ready to start your own
project with this setup!

Subjects
--------
* [Why? A little background information](#why-a-little-background-information)
* [Requirements for setting up this work environment](#requirements-for-setting-up-this-work-environment)
* [Project structure](#project-structure)
* [Drupal to serve JSON Content](#drupal-to-serve-json-content)
* [Phalcon](#phalcon)
    * Create a scalable INI configuration layer
    * Using a JSON Model to fetch data from Drupals generated JSON files
* [Optional scaling of your application](#optional-scaling-of-your-application)
    * Multilingual support

Why? A little background information
------------------------------------
The reason I'm writing this reading is because this setup really helped me to deliver an application
that passed all given requirements. It is no secret that Drupal struggles with performance, especially
for larger websites that generate a lot of traffic and contain even more data.

After building our first application in Phalcon, I was very satisfied by it's fast page loads and
overall performance. Phalcon has a C extension installed as Apache module which makes it the self titled
'fastest PHP Framework'.

When delivering a CMS we're always implementing Drupal and our clients are very happy with this CMS.
The 'problem' is that Drupal doesn't give us the performance Phalcon provides us with. The bootstrap is
just too slow and it's not always the best solution to implement caching layers like Varnish.

And the same way we used Flash as front-end in our past, or using mobile devices in the present. Drupal
will never fail for what it is does best, namely, content management. This is why  we are where we are.
Using Phalcon as our advanced MVC framework filled with content served in our client friendly Drupal CMS.

Requirements for setting up this work environment
-------------------------------------------------
We are ready and set to set up our environment so we can create some awesome code. Before implementing
custom code, we need to download our packages and set up a local environment.

For our Drupal setup we need a Drupal 7 release (used 7.26 for this example, so I would advice this
version or higher) without any necessary contrib modules

For a normal setup it will probably happen that you will use the 'views' module to have more data control,
but it's not necessary for our basic setup.

For Phalcon I've used version 1.2.6. I don't know if the newer 1.3.x release will break anything but you
can always play around with the Apache module to change the version while developing.

The only external library we need is Httpful, a user friendly REST library (http://phphttpclient.com/).
You can extract the contents within a 'Htppful' folder that is placed in app/libraries in our defined
project structure below.

Project structure
-----------------
The project setup I used was divided in the following folder structure:

* web (filled with Phalcon code)
    * public (js/css/images folders & the main index.php file)
    * app (models/views/controllers/config/libraries/classes folders)
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

After setting up this structure we should be set to start adding some custom code to our project so we
can actually connect our 2 platforms.

Drupal to serve JSON Content
----------------------------
The first thing we need in Drupal is a custom module te provide JSON data. Normaly you would create a 'custom'
folder in the sites/all/modules directory with a new 'json_content' folder for our module. The info file
(json_content.info) looks something like this:
```
name = JSON Content
description = Provides functionality to render content as JSON files
package = Custom
core = 7.x
version = 7.0.1
```

The following contents are set in the json_content.module file. I've set the private folder (located at
admin/config/media/file-system) to sites/default/files/private which is needed to use the private uri:
```php
define("JSON_CONTENT_FILE_URI", "private://json_content.json");

/**
 * Cronjob for saving file
 */
function json_content_cron()
{
    _json_content_save_files();
}

/**
 * Save file on insert
 */
function json_content_node_insert()
{
    _json_content_save_files();
}

/**
 * Save file on update
 */
function json_content_node_update()
{
    _json_content_save_files();
}

/**
 * Get content from other modules & save data in JSON file like a boaws
 *
 * @see json_content_cron
 * @see json_content_node_insert
 * @see json_content_node_update
 * @see json_content_node_delete
 */
function _json_content_save_files()
{
    $content = module_invoke_all("json_content_add");
    file_unmanaged_save_data(json_encode($content), JSON_CONTENT_FILE_URI, FILE_EXISTS_REPLACE);
}
```

As you can see, the module provides a hook to add new data for the JSON output. On every insert, update or
cronjob, the file will be overridden with the latest changes.

The use the new hook, we can create a custom module. Let's say it's named 'subscribers' (create a Content
Type named 'Subscriber' without any custom fields), with the following lines of code:
```php
/**
 * @see _json_content_save_files
 */
function subscribers_json_content_add()
{
    return array(
        'subscribers' => _subscribers_get_all()
    );
}

/**
 * @return mixed
 */
function _subscribers_get_all()
{
    $query = db_select("node", "n");
    $query->addField("n", "nid");
    $query->addField("n", "title", "name");
    $query->addField("n", "created");
    $query->condition("n.type", "subscriber", "=");
    return $query->execute()->fetchAllAssoc('nid');
}
```

We've now created a unique key for the subscribers within the JSON file and we are free to create a
structure for each key we add in different modules. In this case it's a simple query that returns 3
fields for each record.

To provide some extra security for our private folder and still make it open for reading. We have a
.htaccess with the following contents:
```
AuthName "Login Credentials"
AuthUserFile /Users/--complete-path-till-cms-folder--/cms/sites/default/files/private/.htpasswd
AuthGroupFile /dev/null
AuthType Basic

require valid-user
```

And the .htpasswd:
```
phalcon_drupal:AAzMaoyXwcGAQ
```

We are now set with a generated JSON file that is secured by a simple authentication layer that we can use
within our Phalcon logic.

Phalcon
-------
If we want Phalcon to retrieve data from our Drupal back-end, we need to save some settings in our Phalcon
config file to make sure our credentials are available in within code logic. I've created a structure that
uses .ini files that are triggered by a domain name. Let's say we're developing on the host phalcon-drupal.dev,
we will create a phalcon_drupal_dev.ini file in our app/config folder with the following contents:
```
;Directories to autoload
[directories]
controllersDir      = '../app/controllers/'
modelsDir           = '../app/models/'
librariesDir        = '../app/libraries/'
classesDir          = '../app/classes/'

;JSON settings for loading Drupals data
[json]
contentUrl          = 'http://playground.dev/phalcon_drupal/cms/sites/default/files/private/json_content.json'
username            = 'phalcon_drupal'
password            = 'phalcon_drupal'
```

In our Phalcon bootstrap (index.php in /public), we can use the following lines of code to load the ini for
our environment & autoload our required folders:
```php
$config = new \Phalcon\Config\Adapter\Ini('../app/config/' . str_replace(array(".", "-"), "_", $_SERVER['HTTP_HOST']) . '.ini');
$loader = new \Phalcon\Loader();
$loader->registerDirs((array)$config->directories)->register();
```

Now that we have our config ready to use, we can create a Model as wrapper around our data. I've named
this Model 'JSONModel' (placed in app/models) and it contains the following contents:
```php
/**
 * Class JSONModel
 */
class JSONModel
{
    /**
     * @var \Phalcon\Config\Adapter\Ini
     */
    private $config;

    /**
     * @var array
     */
    private $data;

    /**
     * @param $config
     */
    public function __construct($config)
    {
        $this->config = $config;
        $this->loadJSON();
    }

    /**
     * Request to load JSON file
     */
    private function loadJSON()
    {
        \Httpful\Bootstrap::init();

        $response = \Httpful\Request::get($this->config->contentUrl)
            ->authenticateWithBasic($this->config->username, $this->config->password)
            ->addHeader("Content-Type", "application/json")
            ->parseWith(function ($body) {
                return json_decode($body, true);
            })
            ->expectsJson()
            ->send();

        $this->data = $response->body;
    }

    /**
     * Retrieve key from data array
     *
     * @param $key
     * @return mixed
     */
    public function get($key)
    {
        return $this->data[$key];
    }
}
```

This is where we use our HttpFul library to create a GET Request to load our data. The public get method
will be available to retrieve the specific data that is stored within a certain key from within Drupal.

We've got a BaseController (placed in app/controllers) that stores a reference to our JSON Model so every
controller that extends our BaseController will be able to use JSON data:
```php
protected $jsonModel;

/**
 * Initialize is for every controller that extends BaseController
 */
public function initialize()
{
    $this->jsonModel = new JSONModel($this->config->json);
}
```

In order to complete the circle, and provide some data for our view, we need a new page. Let's say we want
to provide data to our homepage, we create a IndexController that retrieves and provides data in just one
simple line of code:
```php
public function indexAction()
{
    $this->view->setVars(array(
        'subscribers' => $this->jsonModel->get('subscribers')
    ));
}
```

This ends up with a simple view (placed in views/index/index.phtml) that loops and renders our data within
some clean custom HTML output:
```php
<div class="subscribers">
    <?php foreach ($subscribers as $subscriber): ?>
        <div class="subscriber">
            <h3>
                <span class="name"><?php print $subscriber['name']; ?></span>
                <span class="timestamp"><?php print $subscriber['created']; ?></span>
            </h3>
        </div>
    <?php endforeach; ?>
</div>
```

Great stuff right? A Model abstraction layer, almost no code in our specific page controller action & HTML
freedom that makes any front-end developer smile!

Optional scaling of your application
------------------------------------
