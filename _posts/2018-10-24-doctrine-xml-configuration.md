---
layout: post
title: How to configure Symfony 4 Doctrine XML mapping
---

Imagine `YourDomain\Entity\Customer` domain object. Define your own mapping first:

```
    orm:
        mappings:
            YourDomain\Entity:
                is_bundle: false
                type: xml
                // this is the location where xml files are located, mutatis mutandis
                dir: '%kernel.project_dir%/../src/Infrastructure/ORM/Mapping'
                prefix: 'YourDomain\Entity'
                alias: YourDomain
```

File name has to match the pattern `[class_name].orm.xml`, in your case `Customer.orm.xml`.
If you have sub-namespaces inside, eg. value object `YourDomain\Entity\ValueObject\Email`,
the file has to be named `ValueObject.Email.orm.xml`.

Example mapping:

```
    <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
                      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                      xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                       https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">
        <entity name="YourDomain\Entity\Customer" table="customer">
            <id name="id" type="integer" column="id">
                <generator strategy="AUTO"/>
            </id>
            <field name="email" type="email" unique="true"/>
            <field name="password" length="72"/>
        </entity>
    </doctrine-mapping>
```
