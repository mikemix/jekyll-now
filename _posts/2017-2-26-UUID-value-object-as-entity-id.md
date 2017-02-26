---
layout: post
title: Using custom Value Objects as Doctrine entity ID
---

It's really easy to apply DDD style ValueObject UUID ID's in your Doctrine entities with [custom Doctrine types](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/cookbook/custom-mapping-types.html) and my tiny library available now at [Github](https://github.com/mikemix/ddd-value-object-id).

> Try not to use annotation mapping in your Doctrine entities. You want your core domain to be free of any infrastructure leaks.

Example domain aggregate root. Notice custom type named _article_id_. We will tell Doctrine how to handle this custom type later on.

```php
<?php

namespace DDD\DomainBundle\Article\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity()
 */
class Article
{
    /**
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="NONE")
     * @ORM\Column(type="article_id")
     */
    private $id;

    public function __construct(ArticleId $id)
    {
        $this->id = $id;
    }

    public function getAggregateId(): ArticleId
    {
        return $this->id;
    }
}
```

`ArticleID` extends from `AggregateRootId` to keep some [common logic](https://github.com/mikemix/ddd-value-object-id/blob/master/src/Entity/AggregateRootId.php) separated.

```php
<?php

namespace DDD\DomainBundle\Article\Entity;

use Mikemix\ValueObjectId\Entity\AggregateRootId;

final class ArticleId extends AggregateRootId
{
}
```

All of this would not be possible without custom Doctrine mapping type. My `AbstractUuidType` will handle all the hard work converting to and from the database, storing UUID optimized as `binary(16)`. All you have to do is implement the `getValueObjectClassName` method and return your Value Object's FQCN.

```php
<?php

namespace DDD\InfrastructureBundle\ORM\Type;

use DDD\DomainBundle\Article\Entity;
use Mikemix\ValueObjectId\ORM\Type\AbstractUuidType;

class ArticleIdType extends AbstractUuidType
{
    const ARTICLE_ID = 'article_id';

    public function getName()
    {
        return static::ARTICLE_ID;
    }

    protected function getValueObjectClassName(): string
    {
        return ArticleId::class;
    }
}
```

Type can be easily registered in Symfony's `app/config/config.yml` file:

```yml
doctrine:
    dbal:
        types:
            article_id: DDD\InfrastructureBundle\ORM\Type\ArticleIdType
```

Everything should be wired correctly, lets try it:

```php
<?php

// persist
$article = new Article(new ArticleId(Uuid::uuid4()));
$em->persist($article);
$em->flush();

// retrieve
$article = $em->find(Article::class, 'a1a55638-674a-4d05-a92f-3cd28549ae6d');
//or
$article = $em->find(Article::class, new ArticleId('a1a55638-674a-4d05-a92f-3cd28549ae6d'));
```

Result:

```
DDD\DomainBundle\Article\Entity\Article Object
(
    [id:DDD\DomainBundle\Article\Entity\Article:private] => DDD\DomainBundle\Article\Entity\ArticleId Object
        (
            [uuid:protected] => a1a55638-674a-4d05-a92f-3cd28549ae6d
        )
)
```

Classess are available via [Composer](https://packagist.org/packages/mikemix/ddd-value-object-id) and can be required as dependency in your `composer.json` file, but better idea would be to copy everything to your project. It's always one dependency less.
