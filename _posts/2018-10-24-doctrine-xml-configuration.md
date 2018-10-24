---
layout: post
title: How to configure Symfony 4 Doctrine XML mapping
---

I've attempted to answer this question lately on [StackOverflow](https://stackoverflow.com/questions/52962708/symfony-4-how-to-implement-doctrine-xml-orm-mapping/52963004#52963004). Astoundingly, the documentation is scarce about this.

Imagine `YourDomain\Customer\Customer` domain object:

```
<?php declare(strict_types=1);

namespace YourDomain\Customer;

class Customer
{
    private $id;
    private $email;
    private $password;

    public function __construct(string $email)
    {
        $this->setEmail($email);
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getEmail(): string
    {
        return $this->email;
    }

    public function setEmail(string $email): void
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException('Not a valid e-mail address');
        }

        $this->email = $email;
    }

    public function getPassword(): string
    {
        return (string)$this->password;
    }

    public function setPassword(string $password): void
    {
        $this->password = $password;
    }
}
```

Define your own mapping first:

```
orm:
  mappings:
    YourDomain\Customer:
      is_bundle: false
      type: xml
      // this is the location where xml files are located, mutatis mutandis
      dir: '%kernel.project_dir%/../src/Infrastructure/ORM/Mapping'
      prefix: 'YourDomain\Customer'
      alias: Customer
```

File name has to match the pattern `[class_name].orm.xml`, in your case `Customer.orm.xml`. If you have sub-namespaces inside, eg. value object `YourDomain\Customer\ValueObject\Email`, the file has to be named `ValueObject.Email.orm.xml`.

Example mapping:

```
<doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                   https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">
    <entity name="YourDomain\Customer\Customer" table="customer">
        <id name="id" type="integer" column="id">
            <generator strategy="AUTO"/>
        </id>
        <field name="email" type="email" unique="true"/>
        <field name="password" length="72"/>
    </entity>
</doctrine-mapping>
```
