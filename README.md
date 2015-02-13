# TicketparkFixturesAutoloadBundle

This Symfony2 bundle ads functionalities to simplify loading Doctrine fixtures.

## Todos
* Add unit tests

## Installation

Add TicketparkImageBundle in your composer.json:

```js
{
    "require": {
        "ticketpark/fixtures-autoload-bundle": "dev-master"
    }
}
```

Now tell composer to download the bundle by running the command:

``` bash
$ php composer.phar update ticketpark/fixtures-autoload-bundle
```

Enable the bundle in the kernel:

``` php
<?php
// app/AppKernel.php

public function registerBundles()
{
    $bundles = array(
        // ...
        new Ticketpark\FixturesAutoloadBundle\TicketparkFixturesAutoloadBundle(),
    );
}
```
## Usage

``` php
<?php

namespace Acme\Bundle\SomeBundle\DataFixtures\ORM;

use Doctrine\Common\Persistence\ObjectManager;
use Ticketpark\FixturesAutoloadBundle\Autoloader\Autoloader;

class LoadCountryData extends AutoLoader
{
    public function load(ObjectManager $manager)
    {
        $data = array(
            array(
                '_reference'    => 'CH',
                'shortCode'     => 'CH',
                'name'          => 'Switzerland'
            ),
            array(
                '_reference'    => 'AT',
                'shortCode'     => 'AT',
                'name'          => 'Austria'
            ),
        );

        $this->autoload($data, $manager);
    }
}
```

In a second fixture class, references will be available based on the entity name and the optional `_reference` value:

``` php
<?php

namespace Acme\Bundle\SomeBundle\DataFixtures\ORM;

use Doctrine\Common\DataFixtures\DependentFixtureInterface;
use Doctrine\Common\Persistence\ObjectManager;
use Ticketpark\FixturesAutoloadBundle\Autoloader\Autoloader;

class LoadUserData extends AutoLoader implements DependentFixtureInterface
{
    public function getDependencies()
    {
        return array(
            'Acme\Bundle\SomeBundle\DataFixtures\ORM\LoadCountryData'
        );
    }
    
    public function load(ObjectManager $manager)
    {
        $data = array(
            array(
                // The string `country_CH` references the element
                // created in the 'Country' entity with 'CH' as its
                // _reference value.
                'country' => $this->getReference('country_CH'),
                'name'    => 'Tom Swissman'
            )
        );

        $this->autoload($data, $manager);
    }
}
```


### Overwriting setter method names
In some cases you might want to override the setter methods. E.q. because your method is named `addCurrency` instead of `addCurrencie` as the autoloader by default would asume. In this case, simply use the additional `setterMethods` parameter:

``` php
<?php

namespace Acme\Bundle\SomeBundle\DataFixtures\ORM;

use Doctrine\Common\Persistence\ObjectManager;
use Ticketpark\FixturesAutoloadBundle\Autoloader\Autoloader;

class LoadCurrencyData extends AutoLoader
{
    public function load(ObjectManager $manager)
    {
        $data = array(
            array(
                'currencies' => array(
                    'USD', 'EUR', 'CHF'
                )
            ),
        );

        $setterMethods = array(
            'currencies' => 'addCurrency'
        );

        $this->autoload($data, $manager, $setterMethods);
    }
}
```

## License
This bundle is under the MIT license. See the complete license in the bundle:

    Resources/meta/LICENSE
