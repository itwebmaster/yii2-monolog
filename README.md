## Yii2 Monolog

Yii2 Monolog allows one to use [Monolog](https://github.com/Seldaek/monolog) easily with algolia. For instructions regarding Monolog itself please refer to the [documentation](https://github.com/Seldaek/monolog/blob/master/doc/01-usage.md).

Table of contents
=================
* [Monolog Usage](https://github.com/Seldaek/monolog/blob/master/doc/01-usage.md)
* [Installation](#installation)
* [Configuration](#configuration)
    * [Handlers](#handlers)
    * [Formatters](#formatters)
    * [Processors](#processors)

## Installation
Require this package, with [Composer](https://getcomposer.org/), in the root directory of your project.

```bash
composer require leinonen/yii2-monolog
```

## Configuration
Configure the `leinonen\Yii2Monolog\MonologTarget` as a Log target in your application config.

Example configuration with a basic StreamHandler and a UidProcessor:
```php
use leinonen\Yii2Monolog\MonologTarget;
use Monolog\Handler\StreamHandler;
use Monolog\Processor\UidProcessor;

...
[
    'components' => [
        ...
        'log' => [
            'targets' => [
                'class' => MonologTarget::class,
                'channel' => 'MyChannel',
                'handlers' => [
                    StreamHandler::class => [
                        'path' => '@app/runtime/logs/someLog.log',
                    ],
                ],
                'processors' => [
                    UidProcessor::class,
                ]
            ],
        ],
    ],
    ...
]
```

### Handlers
The package supports all official and 3rd party handlers for Monolog. It uses `leinonen\Yii2Monolog\CreationStrategies\ReflectionStrategy` by default in background to figure out the config values which the handler is to be constructed with. The handlers are defined with a config key `handlers` in the Monolog configuration. All the handlers are resolved through Yii's DI container making it easier to implement your own custom handlers.

Example handler configuration with a stack of two handlers:

```php
[
    ...
    'log' => [
        'targets' => [
            'class' => MonologTarget::class,
            'handlers' => [
                SlackbotHandler::class => [
                    'slackTeam' => 'myTeam',
                    'token' => 'mySecretSlackToken',
                    'channel' => 'myChannel',
                ],
                RotatingFileHandler::class => [
                    'path' => '@app/runtime/logs/myRotatinglog.log',
                    'maxFiles => 10,
                ]
            ]
        ]
    ]
]
```

You can find the available handlers from the [Monolog\Handler namespace](https://github.com/Seldaek/monolog/tree/master/src/Monolog/Handler)

#### Yii2 specific handlers
The package also provides a specific creation strategies for couple of handlers to help integrating with Yii2.

##### StreamHandler
The `path` config value is resolved through Yii's `getAlias()` method making it possible to use aliases such as `@app` in the config.

### Formatters
The package supports all official and 3rd party formatters for Monolog. It uses `leinonen\Yii2Monolog\CreationStrategies\ReflectionStrategy` by default in background to figure out the config values which the formatter is to be constructed with. All the formatters are resolved through Yii's DI container making it easier to implement your own custom formatters.

It's possible to configure a custom formatter for each handler. The formatter is configured with a `formatter` key in the handler's config array:

```php
'handlers' => [
    StreamHandler::class => [
        'path' => '@app/runtime/logs/myLog.log',
        'formatter' => [
            LineFormatter::class => [
                'format' => "[%datetime%] %channel%.%level_name%: %message% %context% %extra%\n",
                'dateFormat' => "Y-m-d\TH:i:sP"
            ]
        ]
    ]
]
```

You can find available formatters from the [Monolog/Formatter](https://github.com/Seldaek/monolog/blob/master/src/Monolog/Formatter) namespace.

### Processors
The package supports all official and 3rd party processors for Monolog. It uses `leinonen\Yii2Monolog\CreationStrategies\ReflectionStrategy` by default in background to figure out the config values which the processor is to be constructed with. All the processors are resolved through Yii's DI container making it easier to implement your own custom processors.

The processors can be defined globally for one target or specifically for a handler. Processors are configured with a `processors` key in the config array with an array of callables:

```php
'log' => [
    'targets' => [
        'class' => MonologTarget::class,
        'processors' => [
            GitProcessor::class,
            function ($record) {
                $record['myCustomData'] = 'test';

                return $record;
            },
        ]
    ]
]
```

Or config to a specific handler:

```php
...
'handlers' => [
    StreamHandler::class => [
    'path' => '@app/runtime/logs/myLog.log',
    'processors' => [
        GitProcessor::class,
        function ($record) {
            $record['myCustomData'] = 'test';

            return $record;
        }
    ]
]
```

You can find available processors from the [Monolog/Processor](https://github.com/Seldaek/monolog/blob/master/src/Monolog/Processor) namespace.
