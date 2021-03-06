
# Basic Usage

`phpSmug` follows the PSR-1, PSR-2 and PSR-4 conventions, which means you can easily use Composer's [autoloading](https://getcomposer.org/doc/01-basic-usage.md#autoloading) to integrate `phpSmug` into your projects.

```php
<?php
// This file is generated by Composer
require_once 'vendor/autoload.php';

// Optional, but definitely nice to have, options
$options = [
    'AppName' => 'My Cool App/1.0 (http://app.com)',
];
$client = new phpSmug\Client('[YOUR_API_KEY]', $options));
$repositories = $client->get('user/[your_username]!albums');
```

For convenience, phpSmug only returns the json_decoded `Response` part of the response from SmugMug.  If you wish to access the full json_decoded response, you can do so with `$client->getResponse();`.

From the `$client` object, you can access to all the SmugMug 2.0 API methods.

# More In-depth Usage Details

## Instantiating the Client

The `phpSmug\Client()` constructor takes two arguments:

- Your API key as a string - **Required**.

    This is required for all interaction with the SmugMug API.  [Applying for an API Key here](https://api.smugmug.com/api/developer/apply).

- An array of options - Optional.

The options you pass here become the default options applied to all requests by default, unless explicitly overwritten elsewhere and can be made up of any combination of the following options:

- `AppName` - The name, version and URL of the application you have built using the phpSmug. There is no required format, but something like `My Cool App/1.0 (http://my.url.com)` would be very useful.

    Whilst this isn't obligatory, it is recommended as it helps SmugMug identify the application that is calling the API in the event one of your users reporting a problem on the SmugMug forums.

- `OAuthSecret` - This is the secret assigned to your API key and is displayed in the Settings tab of the SmugMug Control Panel. If no secret is displayed, select "change" next to the API key your application will use and click "save". A secret will be generated for you.

    An OAuthSecret is required for all access to the SmugMug API that requires authentication.

- `_verbosity` - Determine how much information you'd like to get from SmugMug by increasing or decreasing the verbosity. Defaults to `2`. See [Optimizing response sizes](https://api.smugmug.com/api/v2/doc/advanced/filters.html#verbosity) for more information.

    This option can be overruled on a per-request basis too.

- `_shorturis` - If this parameter is set, the URI section of the response object will be trimmed down so that each entry is just a key-value pair of name to URI. This removes all metadata about the URI from the response. See [Optimizing response sizes](https://api.smugmug.com/api/v2/doc/advanced/filters.html#shorturis) for more information.

    This option can be overruled on a per-request basis too.

- `api_version` - The API version you wish to use. This defaults to `v2` as this is the only version of the API this version of phpSmug is compatible with.  This is really only for "future proofing".

Additionally, you can pass any [Guzzle request option](http://docs.guzzlephp.org/en/latest/request-options.html) though `debug` and `proxy` are probably the only options you may need to set.


## Interacting with the SmugMug API

Once you've instantiated an instance of the `phpSmug\Client`, you can use the simplified [Guzzle HTTP method calls](http://guzzle.readthedocs.org/en/latest/quickstart.html#sending-requests) to interact with the API.

**Note:** phpSmug does not currently support asynchronous requests, though now we rely on Guzzle, this shouldn't be too hard to implement in future.

### GETting Information.

To get information about a user, gallery, folder or image, use

```php
<?php
$client->get($object, $options)
```

If you are not authenticated, you will only be able to access public information.


**The $object - Required**

The object, referenced by `$object` in this and the next examples, is the user, image, album or folder [object identifier](https://api.smugmug.com/api/v2/doc/pages/concepts.html#object-identifiers) you wish to query or modify.

The object can be specified in a number of ways:

- Long form, as SmugMug documents and returns in all API responses:

  ```php
  <?php
  $client->get('/api/v2/user/username!profile');
  ```

- Short form, that is without the `/api/v2/` part:

  ```php
  <?php
  $client->get('user/username!profile');
  ```

- Very short form, for the [special `!authuser` and `!siteuser`](https://api.smugmug.com/api/v2/doc/reference/user.html):

  ```php
  <?php
  $client->get('!authuser');
  ```

You can additionally pass [filters](https://api.smugmug.com/api/v2/doc/advanced/filters.html) in the `$object` path:

```php
<?php
$client->get('user/username!profile?_filter=BioText,Facebook&_filteruri=User');
```

... [expansions](https://api.smugmug.com/api/v2/doc/advanced/expansions.html):

```php
<?php
$client->get('/api/v2/user/username?_expand=UserProfile')
```

... or perform [multi-get](https://api.smugmug.com/api/v2/doc/advanced/multi-get.html) queries:

```php
<?php
$client->get('/api/v2/user/username1,username2?_filteruri=UserProfile');
```

The filters and expansions can also be passed in the `$options` instead if you prefer.

**The GET $options - Optional**

When querying the SmugMug API, you can optionally limit or increase the information SmugMug returns by passing additional optional options to each query.  In the case of `_verbosity` and `_shorturis`, these options overrule those set when instantiating the client.

You can also use the `$options` array to pass any filters or expansions that use query parameters, like `_filter` or `_expand`. For example,

```php
<?php
$options = [
    '_filter' => ['BioText', 'CoverImage'],
    '_filteruri' => ['User'],
    '_shorturis' => true,
    '_verbosity' => 2,
]
$client->get('user/username!profile', $options);
```

If you use the `_expand` option without setting `_expandmethod` to `inline`, SmugMug returns the expansions in their own object indexed by the expanded URI. To make things easier, phpSmug appends this object to the response object.

You can configure expansions using the `_config` option. You can pass this to phpSmug as part of the `$options` array as an array or JSON encoded string. If an array is received, phpSmug will JSON encode it for you else it'll assume it has already been JSON encoded.

### Making Changes

All changes to objects on SmugMug need to be made using the `POST`, `PUT`, `PATCH` or `DELETE` HTTP methods and you can do so as follows:

- `PATCH` - Used to modify one or more data fields of an existing object, like changing the title of an image.

  ```php
  <?php
  $client->patch($object, $options);
  ```

- `PUT` - Used to edit **all** data fields of an existing object. This will replace all field with the information in this request.

  ```php
  <?php
  $client->put($object, $options);
  ```

- `POST` - Used for creating new objects, and for changes that aren't as simple as directly editing a data field, like creating albums and folders, rotating and cropping images, and moving images between galleries.

  ```php
  <?php
  $client->post($object, $options);
  ```

- `DELETE` - Used for deleting objects.

  ```php
  <?php
  $client->delete($object);
  ```

You can find more information about each of these methods at https://api.smugmug.com/api/v2/doc/tutorial/making-changes.html .

**The $object - Required**

The object, referenced by `$object` when making changes, is the user, image, album or folder [object identifier](https://api.smugmug.com/api/v2/doc/pages/concepts.html#object-identifiers) you wish to modify and is accepted in all of the same forms as detailed above.

**The PATCH, PUT, POST $options - Required**

Unlike the GET options, the $options passed to the PATCH, PUT, and POST requests is required as without it, SmugMug won't know what changes you wish to make to the object.

The options you pass are the "Owner-writeable" field for each object type as defined by SmugMug.  For example, these [album fields](https://api.smugmug.com/api/v2/doc/reference/album.html) or these [image fields](https://api.smugmug.com/api/v2/doc/reference/image.html).  The full list of "Owner-writeable" field for each object type can be obtained by querying the `OPTIONS` for the object.


### Probing the API

You can use the OPTIONS HTTP method to find out what other methods an endpoint supports and what parameters those methods accept.

You can query a particular object with:

```php
<?php
$client->options($object);
```

If you set `_verbosity` to `3` in any query, this information will be returned in the response from SmugMug but not made immediately available within the response from that particular call.  You can however grab the information from the raw response using `$client->getResponse()->Options;`. For example:

```php
<?php
$client->get($object, array('_verbosity' => 3));
$client->getResponse()->Options;
```
