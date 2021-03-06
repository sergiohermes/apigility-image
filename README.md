aqilixapi-image
===============

#Images API Handler
This module is purposed to handle image for an API (create/upload, read, update, delete). It use MySQL with Doctrine ORM. This module is extendable to use another database adapter.

Currently this module has support these resources with required OAuth2 Authentication

- POST  /v1.0/image
- GET   /v1.0/image/id
- PATCH /v1.0/image/id
- DELETE  /v1.0/image/id

To retrieve the access token, you can use this resource `POST /oauth` by use some params:

- grant_type
- client_secret
- client_id
- username
- password

Dependencies
------------
- [doctrine/doctrine-orm-module](https://packagist.org/packages/doctrine/doctrine-orm-module)
- [zfcampus/zf-oauth2-doctrine](https://packagist.org/packages/zfcampus/zf-oauth2-doctrine)

Installation
------------
This is a ZF2/Apigility module, so to use it on your ZF2/Apigility project, need to add `repositories` and `require` on `composer.json`. 

```
  "require": {
    .
    .
    .
    "aqilixapi/image": "1.1"
  }
```

Run `composer update` then enable the module on `config/application.config.php`

```
return array(
    'modules' => array(
       .
       .
       .
       'AqilixAPI\\Image', 
       'ZF\\OAuth2\\Doctrine',
       'DoctrineDataFixtureModule'
    )
)
```


Configuration
-------------
Because of this module use `doctrine/doctrine-orm-module`, we just need to configure database credential using configuration file. I prepare the config file here `config/doctrine.local.php.dst`. Just copy this file to `config/autoload/doctrine.local.php` and change the `connection params` 

```
    'connection' => array(
        'orm_default' => array(
            'driverClass' => 'Doctrine\\DBAL\\Driver\\PDOMySql\\Driver',
            'params' => array(
                'host'     => '127.0.0.1',
                'dbname'   => 'apigility',
                'user'     => 'apigility',
                'password' => 'apigility',
            ),
        ),
    ),
```

After that, we need to configure the `image path`, `thumbnail path`, `original file path` and `Asset Manager path`. The configuration file is here `config/aqilixapi.image.local.php.dist`. Copy this file to `config/autoload/aqilixapi.image.local.php` and adjust these configuration

```
    'asset_manager' => array(
        'resolver_configs' => array(
            'paths' => array(
                'data/upload',
            ),
        ),
    ),
    'images' => array(
        'asset_manager_resolver_path' => 'data/upload',
        'target' => 'data/upload/images/img',
        'thumb_path' => 'data/upload/images/thumbs',
        'ori_path'   => 'data/upload/images/ori',
    ),

```

Make sure those paths are exists and writeable by `Web Server`, but if you just use `PHP built in web server` for development you don't need to change their permissions.




OAuth2
----------

To enable `OAuth2` Authentication, just copy default configuration files `(config/oauth2.doctrine-orm.local.php.dist, config/oauth2.local.php.dist)` to `config/autoload/oauth2.doctrine-orm.local.php` and `config/autoload/oauth2.local.php`.

We also able to configure `authorization` based on `Scope`. Currently ACL by `Scope` just supported by `Client Credentials` **Grant Type**. This is caused by limitation from `ZF-OAUTH2`. To add another **Grant Type** we should extend the `ZF-OAUTH2` code. For configuration, just add configuration mentioned above (`config/autoload/aqilixapi.image.local.php`)

```
    'authorization' => array(
        'scopes' => array(
            'post' => array(
                'resource' => 'AqilixAPI\Image\V1\Rest\Image\Controller::collection',
                'method' => 'POST',
            ),
            'update' => array(
                'resource' => 'AqilixAPI\Image\V1\Rest\Image\Controller::entity',
                'method' => 'PATCH',
            ),
            'delete' => array(
                'resource' => 'AqilixAPI\Image\V1\Rest\Image\Controller::entity',
                'method' => 'DELETE',
            )
        )
    ),
```

Define `Scope` name as key, and define `resource` and `method` wanna be authorized. 


Database
--------
This module use a tables `image`, `user` and another tables for `OAuth2`. Currently it use `MySQL`, but you can change it based on your need easily as long as the database is supported by `Doctrine ORM`. If you have follow instructions above, it mean just remain creating the database table.

To do that just run this command from `app skeleton` working directory

```
vendor/bin/doctrine-module orm:schema-tool:create
```

Table will be created and if you wanna try the API with sample data. I have prepare them on the source code. Please run this command 


```
vendor/bin/doctrine-module data-fixture:import
```

Then run the API

```
php -S 0.0.0.0:8080 -t public public/index.php
```

Example
-------
Here are some screenshots I made while trying the API. I use `Postman Chrome Extension` as `REST Client`. You can use the same data with the screenshots, because I make it same in `Data Fixtures`.

####Request Access Token From OAuth Using Credential, Client ID and Client Secret
![Request Access Token From OAuth Using Credential, Client ID and Client Secret](https://github.com/aqilix/apigility-image/blob/master/media/01-request-oauth2-access-token.png)

**Then use the access token on `Authorization Header` while send `Request` to API**

####Upload Image Using POST Method![Upload Image Using POST Method](https://github.com/aqilix/apigility-image/blob/master/media/02-uploading-image-use-post-method.png)

####Retrieve The Uploaded Image Using GET Method![Retrieve The Uploaded Image Using GET Method](https://github.com/aqilix/apigility-image/blob/master/media/03-retrieving-image-use-get-method.png)

####Update Image Using PATCH Method![Update Image Using PATCH Method](https://github.com/aqilix/apigility-image/blob/master/media/04-updating-image-use-patch-method.png)

####Retrieve Images Collection Using GET Method![Retrieve Images Collection Using GET Method](https://github.com/aqilix/apigility-image/blob/master/media/05-retrieving-images-01-using-get-method.png)

![Retrieve Images Collection Using GET Method](https://github.com/aqilix/apigility-image/blob/master/media/06-retrieving-images-02-using-get-method.png)

#####Delete Image Using DELETE Method![Delete Image Using DELETE Method](https://github.com/aqilix/apigility-image/blob/master/media/07-deleting-images-using-del-method.png)
