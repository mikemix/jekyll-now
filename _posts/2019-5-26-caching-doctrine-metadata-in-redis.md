---
layout: post
title: Handling Doctrine metadata cache in Redis
---

I was recently struggling with putting Doctrine's metadata cache in Redis. It was necessary to avoid CPU spikes by preventing requests hitting the backend when there was no cache present yet for the new schema introduced by the deployed application version.
To do this I had to introduce a new Symfony parameter called **SCHEMA_VERSION**:
```yaml
parameters:
  # Most import variable :) If the schema did change
  # One has to bump version number so Redis can pick it up
  SCHEMA_VERSION: 5
```

And reconfigure default caching service to use this parameter as a cache prefix:

```php
services:
    doctrine.system_cache_provider:
        class: Base\Infrastructure\Doctrine\Cache\ExpirableCacheProvider
        public: false
        arguments: ['@doctrine.system_cache_pool']
        calls:
            - {method: setNamespace, arguments: ['dcr_schema_%SCHEMA_VERSION%_']}
```

I also replaced default class with my `ExpirableCacheProvider`, since the default `CacheProvider`
caches the metadata without any lifetime. I didn't want my Redis to fill up with trash 
so I put a hardcoded 30 day lifetime inside my implementation:

```php
<?php

declare(strict_types=1);

namespace Base\Infrastructure\Doctrine\Cache;

use Doctrine\Common\Cache\CacheProvider;
use Psr\Cache\CacheItemPoolInterface;
use Symfony\Component\Cache\PruneableInterface;
use Symfony\Component\Cache\ResettableInterface;
use Symfony\Contracts\Service\ResetInterface;

final class ExpirableCacheProvider extends CacheProvider implements PruneableInterface, ResettableInterface
{
    /**
     * 30 days
     */
    private const LIFETIME_SECONDS = 2592000;

    private $pool;

    public function __construct(CacheItemPoolInterface $pool)
    {
        $this->pool = $pool;
    }

    public function prune()
    {
        return $this->pool instanceof PruneableInterface && $this->pool->prune();
    }

    public function reset()
    {
        if ($this->pool instanceof ResetInterface) {
            $this->pool->reset();
        }
        $this->setNamespace($this->getNamespace());
    }

    protected function doFetch($id)
    {
        $item = $this->pool->getItem(rawurlencode($id));

        return $item->isHit() ? $item->get() : false;
    }

    protected function doContains($id)
    {
        return $this->pool->hasItem(rawurlencode($id));
    }

    protected function doSave($id, $data, $lifeTime = 0)
    {
        $item = $this->pool->getItem(rawurlencode($id));
        $item->expiresAfter(self::LIFETIME_SECONDS);

        return $this->pool->save($item->set($data));
    }

    protected function doDelete($id)
    {
        return $this->pool->deleteItem(rawurlencode($id));
    }

    protected function doFlush()
    {
        $this->pool->clear();
    }

    protected function doGetStats()
    {
    }
}
```

Finally, I made a console command which was introduced the deployment pipeline. This command stores the current version number
inside Redis and compares its previous value with the current one. If they do not match, Doctrine's cache clear command is executed
so Redis is filled with a fresh copy of the new metadata:

```php
<?php

declare(strict_types=1);

namespace Base\Infrastructure\Symfony\Command;

use Psr\Cache\CacheItemPoolInterface;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\ArrayInput;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Style\SymfonyStyle;
use Symfony\Component\Console\Tests\Fixtures\DummyOutput;
use Symfony\Component\DependencyInjection\ParameterBag\ParameterBagInterface;

final class SchemaCacheClearCommand extends Command
{
    private const CACHE_KEY_SCHEMA = 'app.schema_version';

    private $parameters;
    private $systemCache;

    public function __construct(ParameterBagInterface $parameters, CacheItemPoolInterface $systemCache)
    {
        parent::__construct();
        $this->parameters = $parameters;
        $this->systemCache = $systemCache;
    }

    protected function configure()
    {
        $this->setName('app:schema-cache-clear')
            ->setDescription('Clear schema cache during the deployment');
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $console = new SymfonyStyle($input, $output);

        $currentVersion = (int) $this->parameters->get('SCHEMA_VERSION');

        $cacheItem = $this->systemCache->getItem(self::CACHE_KEY_SCHEMA);
        if ($cacheItem->isHit()) {
            $shouldClear = $currentVersion !== (int) $cacheItem->get();
        } else {
            $shouldClear = true;
        }

        if ($shouldClear) {
            $console->caution('New schema detected. Populating the cache…');
            $this->clearMetadataCache();
            $console->success('Cache refreshed');
            $this->systemCache->save($cacheItem->set($currentVersion));
        } else {
            $console->success('Cache schema fresh');
        }
    }

    private function clearMetadataCache(): void
    {
        $app = $this->getApplication()->find('doctrine:cache:clear-metadata');
        $app->run(new ArrayInput([]), new DummyOutput());
    }
}
```

Having cache in Redis for each version guarantees no request will result in a cache miss. 
Should there be some issues with the deployment the app will continue to work smoothly even in case of a rollback.


