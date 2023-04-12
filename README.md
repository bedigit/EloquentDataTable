# EloquentDataTable [![Build status](https://travis-ci.org/LiveControl/EloquentDataTable.svg?branch=master)](https://travis-ci.org/LiveControl/EloquentDataTable)
Eloquent compatible DataTable plugin for server side ajax call handling.

If you are familiar with eloquent and would like to use it in combination with [datatables](https://www.datatables.net/) this is what you are looking for.

## Usage

### Step 1: Install through composer
```composer require livecontrol/eloquent-datatable```

### Step 2: Add DataTables javascript and set it up
For more information check out the [datatables manual](http://datatables.net/manual/index).
Make sure you have [csrf requests](http://laravel.com/docs/master/routing#csrf-protection) working with jquery ajax calls.
```javascript
var table = $('#example').DataTable({
  "processing": true,
  "serverSide": true,
  "ajax": {
    "url": "<url to datatable route>",
    "type": "POST"
  }
});
```

### Step 3: Use it
```php
$users = new Models\User();
$dataTable = new LiveControl\EloquentDataTable\DataTable($users, ['email', 'firstname', 'lastname']);
echo json_encode($dataTable->make());
```

### Optional step 4: Set versions of DataTables plugin.
Just initialize the DataTable object as you would normally and call the setVersionTransformer function as in the following example (for version 1.09 of DataTables):
```php
$dataTable->setVersionTransformer(new LiveControl\EloquentDataTable\VersionTransformers\Version109Transformer());
```
By default the plugin will be loading the transformer which is compatible with DataTables version 1.10.

## Examples

- [Basic setup](#basic-example)
- [Combining columns](#combining-columns)
- [Using raw column queries](#using-raw-column-queries)
- [Return custom row format](#return-custom-row-format)
- [Showing results with relations](#showing-results-with-relations)

### Basic example
```php
use LiveControl\EloquentDataTable\DataTable;

class UserController {
  ...
  public function datatable()
  {
    $users = new User();
    $dataTable = new DataTable(
      $users->where('city', '=', 'London'),
      ['email', 'firstname', 'lastname']
    );
    
    return $dataTable->make();
  }
}
```
In this case we are making a datatable response with all users who live in London.

### Combining columns
If you want to combine the firstname and lastname into one column, you can wrap them into an array.
```php
use LiveControl\EloquentDataTable\DataTable;

class UserController {
  ...
  public function datatable()
  {
    $users = new User();
    $dataTable = new DataTable(
      $users,
      ['email', ['firstname', 'lastname'], 'city']
    );
    
    return $dataTable->make();
  }
}
```
### Using raw column queries
Sometimes you want to use custom sql statements on a column to get specific results,
this can be achieved using the `ExpressionWithName` class.
```php
use LiveControl\EloquentDataTable\DataTable;
use LiveControl\EloquentDataTable\ExpressionWithName;

class UserController {
  ...
  public function datatable()
  {
    $users = new User();
    $dataTable = new DataTable(
      $users,
      [
        'email',
        new ExpressionWithName('`id` + 1000', 'idPlus1000'),
        'city'
      ]
    );
    
    return $dataTable->make();
  }
}
```

### Return custom row format
If you would like to return a custom row format you can do this by adding an anonymous function as an extra argument to the make method.
```php
use LiveControl\EloquentDataTable\DataTable;

class UserController {
  ...
  public function datatable()
  {
    $users = new User();
    $dataTable = new DataTable($users, ['id', ['firstname', 'lastname'], 'email', 'city']);
    
    $dataTable->setFormatRowFunction(function ($user) {
      return [
        $user->id,
        '<a href="/users/' . $user->id . '">' . $user->firstnameLastname . '</a>',
        '<a href="mailto:' . $user->email . '">' . $user->email . '</a>',
        $user->city,
        '<a href="/users/delete/' . $user->id . '">&times;</a>'
      ];
    });
    
    return $dataTable->make();
  }
}
```


### Showing results with relations
```php
use LiveControl\EloquentDataTable\DataTable;

class UserController {
  ...
  public function datatable()
  {
    $users = new User();
    $dataTable = new DataTable(
    	$users->with('country'),
    	['name', 'country_id', 'email', 'id']
    );
    
    $dataTable->setFormatRowFunction(function ($user) {
    	return [
    		'<a href="/users/'.$user->id.'">'.$user->name.'</a>',
    		$user->country->name,
    		'<a href="mailto:'.$user->email.'">'.$user->email.'</a>',
    		'<a href="/users/delete/'.$user->id.'">&times;</a>'
    	];
    });
    
    return $dataTable->make();
  }
}
```


# Omnipay: PayU

**PayU driver for the Omnipay PHP payment processing library**

[Omnipay](https://github.com/thephpleague/omnipay) is a framework agnostic, multi-gateway payment
processing library for PHP 5.3+. This package implements PayU Online Payment Gateway support for Omnipay.

PayU REST API 2.1 [documentation](http://developers.payu.com/en/restapi.html)

This implementation uses OAuth 2

## Installation

Omnipay is installed via [Composer](http://getcomposer.org/). To install, simply add it
to your `composer.json` file:

```json
{
    "require": {
        "bileto/omnipay-payu": "~0.1.1"
    }
}
```
## TL;DR
```php
<?php
require 'vendor/autoload.php';

use Omnipay\PayU\GatewayFactory;

$dotenv = new Dotenv\Dotenv(__DIR__);
$dotenv->load();

// default is official sandbox
$posId = isset($_ENV['POS_ID']) ? $_ENV['POS_ID'] : '300046';
$secondKey = isset($_ENV['SECOND_KEY']) ? $_ENV['SECOND_KEY'] : '0c017495773278c50c7b35434017b2ca';
$oAuthClientSecret = isset($_ENV['OAUTH_CLIENT_SECRET']) ? $_ENV['OAUTH_CLIENT_SECRET'] : 'c8d4b7ac61758704f38ed5564d8c0ae0';

$gateway = GatewayFactory::createInstance($posId, $secondKey, $oAuthClientSecret, true);

try {
    $orderNo = '12345677';
    $returnUrl = 'http://localhost:8000/gateway-return.php';
    $description = 'Shopping at myStore.com';

    $purchaseRequest = [
        'customerIp'    => '127.0.0.1',
        'continueUrl'   => $returnUrl,
        'merchantPosId' => $posId,
        'description'   => $description,
        'currencyCode'  => 'PLN',
        'totalAmount'   => 15000,
        'exOrderId'     => $orderNo,
        'buyer'         => (object)[
            'email'     => 'test@example.com',
            'firstName' => 'Peter',
            'lastName'  => 'Morek',
            'language'  => 'pl'
        ],
        'products'      => [
            (object)[
                'name'      => 'Lenovo ThinkPad Edge E540',
                'unitPrice' => 15000,
                'quantity'  => 1
            ]
        ],
        'payMethods'    => (object) [
            'payMethod' => (object) [
                'type'  => 'PBL', // this is for card-only forms (no bank transfers available)
                'value' => 'c'
            ]
        ]
    ];

    $response = $gateway->purchase($purchaseRequest);

    echo "TransactionId: " . $response->getTransactionId() . PHP_EOL;
    echo 'Is Successful: ' . (bool) $response->isSuccessful() . PHP_EOL;
    echo 'Is redirect: ' . (bool) $response->isRedirect() . PHP_EOL;

    // Payment init OK, redirect to the payment gateway
    echo $response->getRedirectUrl() . PHP_EOL;
} catch (\Exception $e) {
    dump((string)$e);
}
```

For custom sandbox payu gateway prepare `.env` file based on `.env-default`.

## Test cards

### Positive authorization 

| Card provider | Card number
|---|---
|VISA | 4010968243274
|VISA | 4006566732412511
|MAESTRO | 5000579348745235
|MAESTRO | 6999631853158960001
|MASTER CARD| 5100052384536818

### Negative authorization 

| Card provider | Card number
|---|---
|VISA | 4000398284279
|VISA | 4000949177144979
|MAESTRO | 5000105018126595
|MAESTRO | 5794651333329448
|MASTER CARD | 5599575752298650

The expiration date of cards range should be valid date range, value CVC / CVV2 (3 random digits).
Sandbox environment doesn't support 3DS.




