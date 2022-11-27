# selective/test-traits

A collection of PHPUnit test traits.

[![Latest Version on Packagist](https://img.shields.io/github/release/selective-php/test-traits.svg)](https://packagist.org/packages/selective/test-traits)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg)](LICENSE)
[![Build Status](https://github.com/selective-php/test-traits/workflows/build/badge.svg)](https://github.com/selective-php/test-traits/actions)
[![Total Downloads](https://img.shields.io/packagist/dt/selective/test-traits.svg)](https://packagist.org/packages/selective/test-traits/stats)


## Requirements

* PHP 8.0+

## Installation

```bash
composer require selective/test-traits --dev
```

## Traits

### MailerTestTrait

Requirements: `composer require symfony/mailer`

DI container setup example:

```php
use Symfony\Component\EventDispatcher\EventDispatcher;
use Symfony\Component\EventDispatcher\EventDispatcherInterface;
use Symfony\Component\Mailer\EventListener\EnvelopeListener;
use Symfony\Component\Mailer\EventListener\MessageListener;
use Symfony\Component\Mailer\EventListener\MessageLoggerListener;
use Symfony\Component\Mailer\Mailer;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mailer\Transport;
use Symfony\Component\Mailer\Transport\TransportInterface;
// ...

return [
    // Mailer
    MailerInterface::class => function (ContainerInterface $container) {
        return new Mailer($container->get(TransportInterface::class));
    },

    // Mailer transport
    TransportInterface::class => function (ContainerInterface $container) {
        $settings = $container->get('settings')['smtp'];

        // smtp://user:pass@smtp.example.com:25
        $dsn = sprintf(
            '%s://%s:%s@%s:%s',
            $settings['type'],
            $settings['username'],
            $settings['password'],
            $settings['host'],
            $settings['port']
        );

        $eventDispatcher = $container->get(EventDispatcherInterface::class);

        return Transport::fromDsn($dsn, $eventDispatcher);
    },

    EventDispatcherInterface::class => function () {
        $eventDispatcher = new EventDispatcher();
        $eventDispatcher->addSubscriber(new MessageListener());
        $eventDispatcher->addSubscriber(new EnvelopeListener());
        $eventDispatcher->addSubscriber(new MessageLoggerListener());

        return $eventDispatcher;
    },
    
    // ...
],
```

**Usage**:

PHPUnit test case:

```php
<?php

namespace App\Test\TestCase;

use PHPUnit\Framework\TestCase;
use Selective\TestTrait\Traits\ContainerTestTrait;
use Selective\TestTrait\Traits\MailerTestTrait;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Email;

final class MailerExampleTest extends TestCase
{
    use ContainerTestTrait;
    use MailerTestTrait;

    public function testMailer(): void
    {
        $mailer = $this->container->get(MailerInterface::class);

        // Send email
        $email = (new Email())
            ->from('hello@example.com')
            ->to('you@example.com')
            ->subject('Time for Symfony Mailer!')
            ->text('Sending emails is fun again!')
            ->html('<p>My HTML content</p>');

        $mailer->send($email);

        $this->assertEmailCount(1);
        $this->assertEmailTextBodyContains($this->getMailerMessage(), 'Sending emails is fun again!');
        $this->assertEmailHtmlBodyContains($this->getMailerMessage(), '<p>My HTML content</p>');
    }
}
```

### HttpTestTrait

```php
<?php

namespace App\Test\TestCase;

use PHPUnit\Framework\TestCase;
use Selective\TestTrait\Traits\ContainerTestTrait;
use Selective\TestTrait\Traits\HttpTestTrait;

class GetUsersTestAction extends TestCase
{
    use ContainerTestTrait;
    use HttpTestTrait;
     
    public function test(): void
    {
        $request = $this->createRequest('GET', '/api/users');
        $response = $this->app->handle($request);
        $this->assertSame(200, $response->getStatusCode());
    }
}
```

Creating a request with query string parameters:

```php
$request = $this->createRequest('GET', '/api/users')
    ->withQueryParams($queryParams);
```

## RouteTestTrait

Requirements: `composer require slim/slim`

Usage examples

```php
<?php

namespace App\Test\TestCase;

use PHPUnit\Framework\TestCase;
use Selective\TestTrait\Traits\ContainerTestTrait;
use Selective\TestTrait\Traits\HttpTestTrait;
use Selective\TestTrait\Traits\RouteTestTrait;

class GetUsersTestAction extends TestCase
{
    use ContainerTestTrait;
    use HttpTestTrait;
    use RouteTestTrait;
     
    public function test(): void
    {
        $request = $this->createRequest('GET', $this->urlFor('get-users'));
        $response = $this->app->handle($request);
        $this->assertSame(200, $response->getStatusCode());
    }
}
```

Creating a request with a named route and query string parameters:

```php
$request = $this->createRequest('GET',  $this->urlFor('get-users'))
    ->withQueryParams($queryParams);
```

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.
