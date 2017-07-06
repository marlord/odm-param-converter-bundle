# FOOBAR

This bundle provides a symfony param converter for the commercetools odm framework - similar to the doctrine param converter. 

Installation
============

Step 1: Download the Bundle
---------------------------

Open a command console, enter your project directory and execute the
following command to download the latest stable version of this bundle:

```console
$ composer require <FOOBAR>
```

This command requires you to have Composer installed globally, as explained
in the [installation chapter](https://getcomposer.org/doc/00-intro.md)
of the Composer documentation.

Step 2: Enable the Bundle
-------------------------

Then, enable the bundle by adding it to the list of registered bundles
in the `app/AppKernel.php` file of your project:

```php
<?php
// app/AppKernel.php

// ...
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = array(
            // ...

            new BestIt\ODMParamConverterBundle\ODMParamConverterBundle(),
        );

        // ...
    }

    // ...
}
```

Step 3: Using
-------------------------

The shortest usage of the param converter is with a standard `id` field. The converter get the repository of the given class (provided by odm)
and fetch the item. Just use your commercetools object as parameter type hint and name your route id field `id`.
Internal, the `findOneBy` method will be executed and throws a `NotFoundException` if not item was found.

Example:
```
// YourController.php

/**
 * Get custom item
 *
 * @param CustomObject $customObject
 * @Route("/custom/{id}")
 */
public function balanceAction(CustomObject $customObject)
{
    // ...
}
```

If the placeholder does not have the same name as the primary key, pass the id option (you have to annotate the param converter now):

```
// YourController.php

/**
 * Get custom item
 *
 * @param CustomObject $customObject
 * @Route("/custom/{your_route_id}")
 * @ParamConverter("customObject", class="Commercetools\Core\Model\CustomObject\CustomObject", options={
 *    "id" = "your_route_id"
 * }) 
 */
public function balanceAction(CustomObject $customObject)
{
    // ...
}
```

You can create a mapping if you have to pass multiple parameters for fetching an item. Just create an key value pair
at the mapping option. The key is your route name (eg. "container_key"), the value your commercetools object field (eg. "container"):

```
// YourController.php

/**
 * Get custom item
 *
 * @param CustomObject $customObject
 * @Route("/custom/{container_key}/{item_key}")
 * @ParamConverter("customObject", class="Commercetools\Core\Model\CustomObject\CustomObject", options={
 *    "mapping": {"container_key": "container", "item_key": "key"}
 * }) 
 */
public function balanceAction(CustomObject $customObject)
{
    // ...
}
```

If your route params are identical to your commercetools objects fields, you can skip defining the mapping. The param converter will do this
as long as you don't have an `id` key on your route or has defined the `id` option:

```
// YourController.php

/**
 * Get custom item
 *
 * @param CustomObject $customObject
 * @Route("/custom/{container}/{key}")
 */
public function balanceAction(CustomObject $customObject)
{
    // ...
}
```

Sometimes, you want to execute a special method of your repository. You can change the repository method easily by setting
the name of your method in the options. The converter will execute the method with the value of `id` as method argument.

```
// YourController.php

/**
 * Get custom item
 *
 * @param CustomObject $customObject
 * @Route("/custom/{id}")
 * @ParamConverter("customObject", class="Commercetools\Core\Model\CustomObject\CustomObject", options={
 *     "repository_method" = "findByMyOwnId",
 * }) 
 */
public function balanceAction(CustomObject $customObject)
{
    // ...
}
```

Naturally, you can mix the repository method option with your mappings. With the following code, the converter will execute
the `findByCriteria`. The values will be passed as one array argument.

```
// YourController.php

/**
 * Get custom item
 *
 * @param CustomObject $customObject
 * @Route("/custom/{container}/{key}")
 * @ParamConverter("customObject", class="Commercetools\Core\Model\CustomObject\CustomObject", options={
 *     "repository_method" = "findByCriteria",
 * }) 
 */
public function balanceAction(CustomObject $customObject)
{
    // ...
}
```

... 

```
// YourController.php

/**
 * Get custom item
 *
 * @param CustomObject $customObject
 * @Route("/custom/{container_key}/{item_key}")
 * @ParamConverter("customObject", class="Commercetools\Core\Model\CustomObject\CustomObject", options={
 *    "repository_method" = "findByCriteria",
 *    "mapping": {"container_key": "container", "item_key": "key"}
 */
public function balanceAction(CustomObject $customObject)
{
    // ...
}
```

But maybe, the repository does not expect an array. As example, the `findByContainerAndKey` of the custom object repository
expect a container string + a key string and not an array:

```
// CustomObjectRepository.php

public function findByContainerAndKey(string $container, string $key)
{
// ...
}
```

For this reason you can pass the `map_method_signature` option. If true, all parameters will be matched with the method arguments.

```
// YourController.php

/**
 * Get custom item
 *
 * @param CustomObject $customObject
 * @Route("/custom/{container}/{key}")
 * @ParamConverter("customObject", class="Commercetools\Core\Model\CustomObject\CustomObject", options={
 *     "repository_method" = "findByContainerAndKey",
 *     "map_method_signature" = true
 * }) 
 */
public function balanceAction(CustomObject $customObject)
{
    // ...
}
```

... 

```
// YourController.php

/**
 * Get custom item
 *
 * @param CustomObject $customObject
 * @Route("/custom/{container_key}/{item_key}")
 * @ParamConverter("customObject", class="Commercetools\Core\Model\CustomObject\CustomObject", options={
 *    "repository_method" = "findByContainerAndKey",
 *    "mapping": {"container_key": "container", "item_key": "key"},
 *    "map_method_signature" = true
 */
public function balanceAction(CustomObject $customObject)
{
    // ...
}
```