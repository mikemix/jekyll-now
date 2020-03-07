---
layout: post
title: Efficient Twig extensions using Lazy Extensions
---

[Lazy Twig extensions](https://symfony.com/doc/current/templating/twig_extension.html#creating-lazy-loaded-twig-extensions)
were introduced in Twig 1.35 / 2.4.4, albeit there're still see so many Symfony apps that don't use them.

One has to be aware that to create the Twig environment, every extension has to be instantiated.
Now multiply that by the number of extensions in your app, some of them with possibly large dependency tree.

For instance, take a look at [Sylius](https://sylius.com/) – popular e-commerce platform based on Symfony 
and its `CheckoutStepsExtension` – it depends on a helper, which depends on two services, both depending on composite 
resolvers, which in turn load another set of their dependencies. 

This extension has to be instantiated every single time Twig is used, even though it's not needed.

So instead of:

```php
final class CheckoutStepsExtension extends \Twig_Extension
{
    private $checkoutStepsHelper;

    public function __construct(CheckoutStepsHelper $checkoutStepsHelper)
    {
        $this->checkoutStepsHelper = $checkoutStepsHelper;
    }

    public function getFunctions(): array
    {
        return [
            new \Twig_Function('sylius_is_shipping_required', [$this->checkoutStepsHelper, 'isShippingRequired']),
            new \Twig_Function('sylius_is_payment_required', [$this->checkoutStepsHelper, 'isPaymentRequired']),
        ];
    }
}
```

we could move dependencies from the extension to a Twig Runtime:

```php
final class CheckoutStepsExtension extends \Twig_Extension
{
    public function getFunctions(): array
    {
        return [
            new \Twig_Function('sylius_is_shipping_required', [CheckoutStepsHelperRuntime::class, 'isShippingRequired']),
            new \Twig_Function('sylius_is_payment_required', [CheckoutStepsHelperRuntime::class, 'isPaymentRequired']),
        ];
    }
}
```

and the Runtime proxy:

```php
final class CheckoutStepsHelperRuntime implements RuntimeExtensionInterface
{
    private $checkoutStepsHelper;

    public function __construct(CheckoutStepsHelper $checkoutStepsHelper)
    {
        $this->checkoutStepsHelper = $checkoutStepsHelper;
    }
    
    public function isShippingRequired(OrderInterface $order): bool
    {
        return $this->checkoutStepsHelper->isShippingRequired($order);
    }

    public function isPaymentRequired(OrderInterface $order): bool
    {
        return $this->checkoutStepsHelper->isPaymentRequired($order);
    }
}
```

The `RuntimeExtensionInterface` interface is just for convenience. It enables Symfony DI's autoconfiguration feature
to tag the class with the `twig.runtime` tag.

Last, but not least I hope, remember to test your Twig extensions. In the next post I'll show you how to do that properly.
