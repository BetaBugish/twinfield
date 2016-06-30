# Twinfield  [![Build Status](https://secure.travis-ci.org/pronamic/twinfield.png?branch=develop)](http://travis-ci.org/pronamic/twinfield)
A PHP library for Twinfield Integration. Developed by [Remco Tolsma](http://remcotolsma.nl/) and [Leon Rowland](http://leon.rowland.nl/) from [Pronamic](http://pronamic.nl/).
Use the Twinfield SOAP Service to have your PHP application communicate directly with your Twinfield account.

---

## Autoloading

The classes follow the PSR2 naming convention.


## Usage

### General Usage Information
Components will have Factories to simplify the request and send process of Twinfield.
Each factory will require just the \Pronamic\Twinfield\Secure\Config() class with
the filled in details.

An example of the usage of the Configuration class.

```php
$config = new \Pronamic\Twinfield\Secure\Config();
$config->setCredentials('Username', 'Password', 'Organization', 'Office');
```

* or, when using OAuth:

```php
$config = new \Pronamic\Twinfield\Secure\Config();
$config->setOAuthParameters('clientID', 'clientSecret', 'returnURL', 'Organization', 'Office', true);
//the true parameter at the end tells the system to automatically redirect to twinfield to login
```



Now, to the current modules

In the following example, we will use the Customer component as showcase. Although 
this will be the method for all components ( including Invoice currently )

Typically it is as follows, if using the Factories

* Add/Edit: Make Object, Make Factory, Give object in Submit method of related factory.
* Retrieve: Make Factory, Supply all required params to respective listAll() and get() methods

#### Add/Edit

Make your Customer object

```php
$customer = new \Pronamic\Twinfield\Customer\Customer();
$customer
	->setID(10666)
	->setName('Leon Rowland')
	->setType('DEB')
	->setWebsite('http://leon.rowland.nl')
	->setEBilling(true)
	->setEBillMail('leon@rowland.nl')
	->setVatCode('VL')
	->setDueDays(10)
	->setCocNumber('12341234');
```

Customers can have addresses associated with them

```php
$customerAddress = new \Pronamic\Twinfield\Customer\CustomerAddress();
$customerAddress
	->setDefault(false)
	->setType('invoice')
	->setField1('Testing field 1')
	->setField2('Testing field 2')
	->setField3('Testing field 3')
	->setPostcode('1212 AB')
	->setCity('TestCity')
	->setCountry('NL')
	->setTelephone('010-12345')
	->setFax('010-1234')
	->setEmail('test@email.com');
```

Assign that address to the customer

```php
$customer->addAddress($customerAddress);
```

Now lets submit it!

```php
use \Pronamic\Twinfield\Customer as TwinfieldCustomer;

// Config object prepared and passed to the CustomerFactory
$customerFactory = new TwinfieldCustomer\CustomerFactory($config);

//$customer = new TwinfieldCustomer\Customer();

// Attempt to send the Customer document
if($customerFactory->send($customer)){
	// Use the Mapper to turn the response back into a TwinfieldCustomer\Customer
	$successfulCustomer = TwinfieldCustomer\Mapper\CustomerMapper::map($customerFactory->getResponse());
}
```

#### Retrieve/Request

You can get all customers or get a single one currently.

```php
use \Pronamic\Twinfield\Customer as TwinfieldCustomer;

// Config object prepared and passed into the CustomerFactory
$customerFactory = new TwinfieldCustomer\CustomerFactory($config);

$customers = $customerFactory->listAll();
```

At the moment, listAll will return an array of just name and short name.

```php

$customer = $customerFactory->get('customerCode', 'office[optional]');
```

The response from get() will be a \Pronamic\Twinfield\Customer\Customer object.

#### Browse

Browse requests allow you to retreive detailed reports about just anything entered into Twinfield. You do so by fetching from a 'browse definition' which are outlined [here](https://c3.twinfield.com/webservices/documentation/#/ApiReference/Request/BrowseData#Example).
```php
use \Pronamic\Twinfield\Secure\Config;

use \Pronamic\Twinfield\Browse\BrowseFactory;
use \Pronamic\Twinfield\Browse\Browse;
use \Pronamic\Twinfield\Browse\Column;

define('TWINFIELD_ADMINISTRATION', '');
define('TWINFIELD_USER', '');
define('TWINFIELD_PASSWORD', '');
define('TWINFIELD_COMPANY', '');

$config = new Config();
$config->setCredentials(TWINFIELD_USER, TWINFIELD_PASSWORD, TWINFIELD_COMPANY, TWINFIELD_ADMINISTRATION);

$browseFactory = new BrowseFactory($config);

$browse = new Browse;
$browse->setCode('200');

$columnCreditor = new Column;
$columnCreditor->setId(1);
$columnCreditor->setField('fin.trs.line.dim2');
$columnCreditor->setLabel('Creditor');
$columnCreditor->setOperator('between');
$columnCreditor->setFrom(20000);
$columnCreditor->setVisible(true);
$browse->addColumn($columnCreditor);

$columnNumber = new Column;
$columnNumber->setId(2);
$columnNumber->setField('fin.trs.head.number');
$columnNumber->setLabel('Trans nr');
$columnNumber->setVisible(true);
$browse->addColumn($columnNumber);

$columnDaybook = new Column;
$columnDaybook->setId(3);
$columnDaybook->setField('fin.trs.head.code');
$columnDaybook->setLabel('Code');
$columnDaybook->setVisible(true);
$browse->addColumn($columnDaybook);

$columnOffice = new Column;
$columnOffice->setId(4);
$columnOffice->setField('fin.trs.head.office');
$columnOffice->setOperator('equal');
$columnOffice->setFrom(TWINFIELD_ADMINISTRATION);
$columnOffice->setVisible(true);
$browse->addColumn($columnOffice);

$browseResult = $browseFactory->send($browse);

// The browse request failed, the API is rather specific about what it accepts.
if(!$browseResult) {
    throw new Exception($browseFactory->getResponse()->getResponseDocument()->saveXML());
}

// The criteria might be incorrect resulting in 0 returned columns
if(!count($browseResult->getLines())) {
    throw new Exception('The result doesn\'t have any lines!');
}

foreach($browseResult->getLines() as $line) {
	echo $line->getField('fin.trs.head.number')->getValue() . "\n";
}
```


#### Notes

Advanced documentation coming soon. Detailing usage without the Factory class. Giving you more control
with the response and data as well as more in-depth examples and usage recommendations.


## Contribute

You can contribute to the development of this project. Try and keep to the way of doing things as
the other 2 components have implemented.

A large requirement is to maintain backwards compatibility so if you have any plans for large
restructure or alteration please bring up in an issue first.

| Component                                                                                                       | get()              | listAll()          | send()             | Mapper             | Namespace                                                                                                               |
| --------------------------------------------------------------------------------------------------------------- | :----------------: | :----------------: | :----------------: | :----------------: | ----------------------------------------------------------------------------------------------------------------------- |
| [Customer](https://c1.twinfield.com/webservices/documentation/#/ApiReference/Masters/Customers)                 | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | [Pronamic/Twinfield/Customer](https://github.com/pronamic/twinfield/tree/develop/src/Pronamic/Twinfield/Customer)       |
| [Sales Invoices](https://c1.twinfield.com/webservices/documentation/#/ApiReference/SalesInvoices)               | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | [Pronamic/Twinfield/Invoice](https://github.com/pronamic/twinfield/tree/develop/src/Pronamic/Twinfield/Invoice)         |
| Transactions: [Purchase](https://c1.twinfield.com/webservices/documentation/#/ApiReference/PurchaseTransactions) [Sale](https://c1.twinfield.com/webservices/documentation/#/ApiReference/SalesTransactions) |                    |                    | :white_check_mark: |                    | [Pronamic/Twinfield/Transaction](https://github.com/pronamic/twinfield/tree/develop/src/Pronamic/Twinfield/Transaction) |
| [Articles](https://c1.twinfield.com/webservices/documentation/#/ApiReference/Masters/Articles)                  |                    |                    |                    |                    | Pronamic/Twinfield/Article                                                                                              |
| [Balance Sheets](https://c1.twinfield.com/webservices/documentation/#/ApiReference/Masters/BalanceSheets)       |                    |                    |                    |                    | Pronamic/Twinfield/BalanceSheet                                                                                         |
| [Suppliers](https://c1.twinfield.com/webservices/documentation/#/ApiReference/Masters/Suppliers)                | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | Pronamic/Twinfield/Supplier                                                                                             |
| [Dimension Groups](https://c1.twinfield.com/webservices/documentation/#/ApiReference/Masters/DimensionGroups)   |                    |                    |                    |                    | Pronamic/Twinfield/Dimension/Group                                                                                      |
| [Dimension Types](https://c1.twinfield.com/webservices/documentation/#/ApiReference/Masters/DimensionTypes)     |                    |                    |                    |                    | Pronamic/Twinfield/Dimension/Type                                                                                       |
| [Offices](https://c1.twinfield.com/webservices/documentation/#/ApiReference/Masters/Offices)                    |                    | :white_check_mark: |                    |                    | Pronamic/Twinfield/Office                                                                                               |
| [Vat types](https://c3.twinfield.com/webservices/documentation/#/ApiReference/Miscellaneous/Finder)             |                    | :white_check_mark: |                    |                    | Pronamic/Twinfield/VatCode                                                                                              |


## Build

*	npm install
*	composer install


## Links

* [Twinfield API Documentation site](https://c1.twinfield.com/webservices/documentation/)
* [Twinfield Library for Python](https://bitbucket.org/vanschelven/twinfield)
* [Using Grunt for PHP](https://chrsm.org/post/using-grunt-for-php/)
* [Using Grunt with PHP Quality Assurance Tools](http://mariehogebrandt.se/articles/using-grunt-php-quality-assurance-tools/)


## Authors

*	[Pronamic](http://pronamic.nl/)
*	[Remco Tolsma](http://remcotolsma.nl/)
*   [Emile Bons](http://www.emilebons.nl)


## License

Copyright 2009-2013 Pronamic.

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program; if not, write to the Free Software
Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA 02111-1307 USA
