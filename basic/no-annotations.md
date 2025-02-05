# Using ORM without Annotations
You can use the ORM engine without annotations. You can either declare a mapping schema via configuration
or use the cycle/schema-builder extension to compile it on the fly via OOP wrappers.

## Examples
There are multiple examples of using the ORM with manually defined mapping schema:
- https://github.com/cycle/docs/blob/master/advanced/manual.md
- https://github.com/cycle/docs/blob/master/advanced/dynamic-schema.md
- https://github.com/cycle/docs/blob/master/advanced/schema-builder.md
- https://github.com/cycle/docs/blob/master/advanced/active-record.md

## Quick Sample
We can demonstrate how to create an ORM instance which maps objects `User` and `Profile` with a `HasOne` relation between them.
Such setup does not require any caching as the schema is defined in code:

```php
<?php declare(strict_types=1);
include 'vendor/autoload.php';

use Spiral\Database;
use Cycle\ORM;
use Cycle\ORM\Schema;

$dbal = new Database\DatabaseManager(
    new Database\Config\DatabaseConfig([
        'default'     => 'default',
        'databases'   => [
            'default' => ['connection' => 'sqlite']
        ],
        'connections' => [
            'sqlite' => [
                'driver'  => Database\Driver\SQLite\SQLiteDriver::class,
                'connection' => 'sqlite:database.db',
            ]
        ]
    ])
);

$orm = new ORM\ORM(new ORM\Factory($dbal), new Schema([
   'user'    => [
      Schema::ENTITY      => User::class,
      Schema::MAPPER      => ORM\Mapper\Mapper::class,
      Schema::DATABASE    => 'default',
      Schema::TABLE       => 'user',
      Schema::PRIMARY_KEY => 'id',
      Schema::COLUMNS     => [
          'id'        => 'id',
          'email'     => 'email',
          'balance'   => 'balance'
      ],
      Schema::TYPECAST    => [
          'id'      => 'int',
          'balance' => 'float'
      ],
      Schema::RELATIONS   => [
          'profile' => [
              Relation::TYPE   => ORM\Relation::HAS_ONE,
              Relation::TARGET => 'profile',
              Relation::SCHEMA => [
                  Relation::CASCADE   => true,
                  Relation::INNER_KEY => 'id',
                  Relation::OUTER_KEY => 'user_id',
              ],
          ]
      ]
  ],
  'profile' => [
      Schema::ENTITY      => Profile::class,
      Schema::MAPPER      => ORM\Mapper\Mapper::class,
      Schema::DATABASE    => 'default',
      Schema::TABLE       => 'profile',
      Schema::PRIMARY_KEY => 'id',
      Schema::COLUMNS     => [
          'id'      => 'id',
          'user_id' => 'user_id',
          'image'   => 'image'
      ],
      Schema::TYPECAST    => [
          'id'      => 'int',
          'user_id' => 'int'
      ],
      Schema::RELATIONS   => []
  ],
]));

print_r($orm->getRepository(User::class)->findOne());
```

> You only need the `cycle/orm` dependency for this example. Note, you cannot access the `profile` relation without explicitly calling it in the Select, use `cycle/proxy-factory` to enable lazy loading. You can find more examples in [orm tests](https://github.com/cycle/orm/tree/master/tests/ORM).
