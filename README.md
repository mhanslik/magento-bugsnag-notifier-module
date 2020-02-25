# Magento 2 Bugsnag Notifier

This module integrates the Bugsnag notifier into Magento's exception handling. Any exception that is left unhandled and eventually caught within the `Magento\Framework\App\Bootstrap::run` method will be notified to Bugsnag.

This works by using two interceptors that are placed before the `launch` and `catchException` methods on any class that implement the `Magento\Framework\AppInterface` interface.

## Configuration

To configure the Bugsnag Notifier you have two options:

1) Use the environment variable: `BUGSNAG_API_KEY`, `BUGSNAG_ENDPOINT`, `BUGSNAG_RELEASE_STAGE`

2) Add the following configuration to: `app/etc/env.php`

```php
return [
    ...
    'bugsnag' => [
        'api_key' => 'YOUR API KEY',
        'endpoint' => 'custom endpoint if required',
        'release_stage' => 'production'
    ]
]
```

The `endpoint`/`BUGSNAG_ENDPOINT` is optional, and only required if you need to notify a different endpoint, such as a self-hosted edition of Bugsnag.

The `release_stage`/`BUGSNAG_RELEASE_STAGE` will default to `production` so is also optional.


## Extending Bugsnag Metadata

If you wish to send additional data to Bugsnag that may help debug your Magento application, there are two events dispatched by this module which can be used to add additionaal metadata.

### Event: `bugsnag_init`
`bugsnag_init` is dispatched when the module first initialises Bugsnag and a session is started. You'll be able to retrive the Bugsnag client by doing: `$observer->getData('client')` which will give you an instance of `\Bugsnag\Client`. From there adding metadata can be done like so:

```php
/** @var \Bugsnag\Client $client */
$client = $observer->getData('client');
$client->setMetaData([
    'app' => [
        'new_property' => 'new_value'
    ]
]);
```

Alternatively you can register a callback which will be executed when an exception occurs to add additional information to the report:

```php
/** @var \Bugsnag\Client $client */
$client = $observer->getData('client');
$client->registerCallback(function (\Bugsnag\Report $report) {
    $report->setMetaData([
        'app' => [
            'new_property' => 'new_value'
        ]
    ])
});
```

### Event: `bugsnag_add_customer_data`
This event allows you add additional customer data to the exception report, you can fetch the current customer data object by calling: `$observer->getData('data')`.

This is an instance of `\Magento\Framework\DataObject`, so you can use the `setData` and `getData` properties to modify the object.

By default the module will only send the customers: ID, customer group, and a boolean whether or not they are logged in.