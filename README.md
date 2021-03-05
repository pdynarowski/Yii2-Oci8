# Yii2-Oci8
This extension is forked Neconix/Yii2-Oci8 to support newer versions yajra/pdo-via-oci8 and PHP 8
Yii2 OCI8 extension which uses well written [pdynarowski/pdo-via-oci8](https://github.com/pdynarowski/pdo-via-oci8) 
with optional full table schema caching. Supported PHP 7.4 and 8.

**Supported**
- Yii 2.x
- yajra/pdo-via-oci8 2.x
- \>= PHP 7.4
- \>= PHP 8

**Installation**

Add to your `composer.json` file:

```
   "require": {
     "pdynarowski/yii2-oci8": "^1.1"
   }
```

And then run `composer update`.

**Yii2 configuration example for an Oracle database**

Yii2 configuration:

```php
$config = [
    ...
    'components' => [
        ...
        'db' => require(__DIR__ . '/db.php'),
        ...
    ]
];
```

Database configuration in `db.php`:

```php
return [
'class' => 'pdynarowski\yii2oci8\Oci8Connection',
        'dsn' => 'oci:dbname=//192.168.0.1:1521/instance_name;charset=UTF8',
        'username' => 'username',
        'password' => 'password',
        'enableSchemaCache' => false,
        'on afterOpen' => function($event) {
            $event->sender->createCommand("ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY-MM-DD HH24:MI:SS'")->execute();
        },
        'schemaCacheDuration' => 0,
        'attributes' => [ PDO::ATTR_PERSISTENT => true ],
        'on afterOpen' => function($event)
        {
            /* @var $schema \neconix\yii2oci8\CachedSchema */
            $event->sender->createCommand("ALTER SESSION SET NLS_DATE_FORMAT = 'YYYY-MM-DD HH24:MI:SS'")->execute();
            $schema = $event->sender->getSchema();
            $schema->schemaCacheDuration = 0; // 0 - never expire
            if (!$schema->isCached) {
                //Rebuild schema cache
                $schema->buildSchemaCache();
            }
        },
        //Defining a cache schema component
        'cachedSchema' => [
            'class' => 'pdynarowski\yii2oci8\CachedSchema',
            'schemaCacheDuration'=>0, //60*60,
        ],
];
```

**Example**

Feel free to use Yii2 `ActiveRecord` methods:

```php
$cars = Car::find()->where(['YEAR' => '1939'])->indexBy('ID')->all();
```

Getting a raw database handler and working with it:

```php
$dbh = Yii::$app->db->getDbh();
$stmt = oci_parse($dbh, "select * from DEPARTMENTS where NAME = :name");
$name = 'NYPD';
oci_bind_by_name($stmt, ':name', $name);
oci_execute($stmt);
...
//fetching result
```

**Caching features**

To enable caching for all tables in a schema add lines below in a database connection configuration `db.php`:

```php
    ...
    //Disabling Yii2 schema cache
    'enableSchemaCache' => false
    
    //Defining a cache schema component
    'cachedSchema' => [
        'class' => 'pdynarowski\yii2oci8\CachedSchema',
        // Optional, default is the current connection schema.
        'cachingSchemas' => ['HR', 'SCOTT'],
        // Optional. This callback must return `true` for a table name if it need to be cached.
        'tableNameFilter' => function ($tableName, $schemaName) {
            //Cache everything but the EMP table from HR and SCOTT schemas
            return $tableName != 'EMP';
        }
    ],
    ...
```

Table schemas saves to the default Yii2 cache component.
To build schema cache after a connection opens:

```php
    'on afterOpen' => function($event) 
    {
        $event->sender->createCommand($q)->execute();

        /* @var $schema \pdynarowski\yii2oci8\CachedSchema */
        $schema = $event->sender->getSchema();

        if (!$schema->isCached)
            //Rebuild schema cache
            $schema->buildSchemaCache();
    },
```