# Yii2-Oci8
This extension is forked Neconix/Yii2-Oci8 to support newer versions yajra/pdo-via-oci8 and PHP 8
Yii2 OCI8 extension which uses well written [pdynarowski/pdo-via-oci8](https://github.com/pdynarowski/pdo-via-oci8) 

**Supported**
- Yii 2.x
- yajra/pdo-via-oci8 3.x
- \>= PHP 8

**Installation**

Add to your `composer.json` file:

```
   "require": {
     "pdynarowski/yii2-oci8": "^2.0"
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
	    $event->sender->createCommand("ALTER SESSION SET NLS_TIMESTAMP_TZ_FORMAT = 'YYYY-MM-DD HH:MI:SS'")->execute();
            $event->sender->createCommand("ALTER SESSION SET NLS_TIMESTAMP_FORMAT = 'YYYY-MM-DD HH24:MI:SS'")->execute();
        }
];
```
