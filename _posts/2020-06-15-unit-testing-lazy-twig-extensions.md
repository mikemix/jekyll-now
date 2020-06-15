---
layout: post
title: Unit testing Twig's Lazy Extensions
excerpt: Last time we wrote a lazy Twig Extension. Today it's time to test it.
---

[Last time](/efficient-twig-extensions/) we wrote a 
[lazy Twig extension](https://symfony.com/doc/current/templating/twig_extension.html#creating-lazy-loaded-twig-extensions)
to speed up Twig initialization time.

Today it's time to test it. There's an excellent helper class called `IntegrationTestCase` in the Twig package.
This class expects a directory containing Twig test cases called "fixtures". 

```php
final class CheckoutStepsExtensionTest extends IntegrationTestCase
{
    /**
     * The directory with test cases
     */
    protected function getFixturesDir(): string
    {
        return __DIR__ . '/Fixtures';
    }
    
    /**
     * Unit tested extensions
     */
    protected function getExtensions(): array
    {
        return [new CheckoutStepsExtension()];
    }
    
    /**
     * Called functions expects an "Order" object.
     * Lets create a helper function will will return an Order.
     */
    protected function getTwigFunctions(): array
    {
        return [
            new TwigFunction('get_test_order', function (): OrderInterface {
                return new Order();
            }),
        ];
    }

    /**
     * Since the runtime depends on its dependencies when created
     * we need to register custom runtime loader which will spawn called runtime.
     * For the sake of simplicity, we will mock the actual helper.
     */
    protected function getRuntimeLoaders(): array
    {
        return [new FactoryRuntimeLoader([
            CheckoutStepsHelperRuntime::class => function (): CheckoutStepsHelperRuntime {
                $helper = $this->prophesize(CheckoutStepsHelper::class);
                $helper->isShippingRequired(Argument::type(Order::class))->willReturn(true);
                $helper->isPaymentRequired(Argument::type(Order::class))->willReturn(true);

                return new CheckoutStepsHelperRuntime($helper->reveal());
            }
        ])];
    }
}
```

The test is complete, lets create some tests inside the "fixtures" directory:

The `fixtures/functions/sylius_is_shipping_required.test` file:

```
--TEST--
"sylius_is_shipping_required" function
--TEMPLATE--
{{ sylius_is_shipping_required(get_test_order()) ? 'yes' : 'no' }}
--DATA--
return []
--EXPECT--
yes
```

Finally the `fixtures/functions/sylius_is_payment_required.test` file:

```
--TEST--
"sylius_is_payment_required" function
--TEMPLATE--
{{ sylius_is_payment_required(get_test_order()) ? 'yes' : 'no' }}
--DATA--
return []
--EXPECT--
yes
```

Run the suite `vendor/bin/phpunit` and we should be GREEN OK! 
We are yellow OK though since the test case runs some weird `testLegacyIntegration` test which gets skipped.

However if we were to change the expectation to "no" we would end up with:

```
Failed asserting that two strings are equal.
--- Expected
+++ Actual
@@ @@
-'no'
+'yes'

/var/www/html/vendor/twig/twig/src/Test/IntegrationTestCase.php:251
/var/www/html/vendor/twig/twig/src/Test/IntegrationTestCase.php:82
```
