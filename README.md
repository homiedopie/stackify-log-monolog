stackify-log-monolog V2
================

[![PHP version](https://badge.fury.io/ph/stackify%2Fmonolog.svg)](http://badge.fury.io/ph/stackify%2Fmonolog)

Monolog handler for sending log messages and exceptions to Stackify.
Monolog >=1.1.0  <2.0.0 is supported.

> For Monolog > 2.0.0, please use the [2.x branch](https://github.com/stackify/stackify-log-monolog/tree/2.x)

Errors and Logs Overview:

http://support.stackify.com/errors-and-logs-overview/

Smarter Errors & Logs for PHP:

http://stackify.com/smarter-errors-logs-php/

Sign Up for a Trial:

http://www.stackify.com/sign-up/

## Installation

Install the latest version with `composer require stackify/monolog "~1.0"`

Or add dependency to `composer.json` file:
```json
"stackify/monolog": "~2.0",
```

If you use [Symfony Monolog Bundle](https://github.com/symfony/MonologBundle) it is best to configure the Stackify handler using the Symfony Dependency Injection configuration files. See examples below.

There are three different transport options that can be configured to send data to Stackify.  Below will show how to implement the different transport options.

### ExecTransport

ExecTransport does not require a Stackify agent to be installed because it sends data directly to Stackify services. It collects log entries in batches, calls curl using the ```exec``` function, and sends data to the background immediately [```exec('curl ... &')```]. This will affect the performance of your application minimally, but it requires permissions to call ```exec``` inside the PHP script and it may cause silent data loss in the event of any network issues. This transport method does not work on Windows. To configure ExecTransport you need to pass the environment name and API key (license key):

PHP:
```php
use Stackify\Log\Transport\ExecTransport;
use Stackify\Log\Monolog\Handler as StackifyHandler;
    
$transport = new ExecTransport('api_key');
$handler = new StackifyHandler('application_name', 'environment_name', $transport);
$logger = new Logger('logger');
$logger->pushHandler($handler);
```
   
Symfony: 
```yml
services:
    stackify_transport:
        class: "Stackify\\Log\\Transport\ExecTransport"
        arguments: ["api_key"]
    stackify_handler:
        class: "Stackify\\Log\\Monolog\\Handler"
        arguments: ["application_name", "environment_name", "@stackify_transport"]
monolog:
    handlers:
        stackify:
            type: service
            id: stackify_handler
```

#### Optional Configuration

**Proxy**
- ExecTransport supports data delivery through proxy. Specify proxy using [libcurl format](http://curl.haxx.se/libcurl/c/CURLOPT_PROXY.html): <[protocol://][user:password@]proxyhost[:port]>
```php
$transport = new ExecTransport($apiKey, ['proxy' => 'https://55.88.22.11:3128']);
```

**Curl path**
- It can be useful to specify ```curl``` destination path for ExecTransport. This option is set to 'curl' by default.
```php
$transport = new ExecTransport($apiKey, ['curlPath' => '/usr/bin/curl']);
```

**Log Server Environment Variables**
- Server environment variables can be added to error log message metadata. **Note:** This will log all 
system environment variables; do not enable if sensitive information such as passwords or keys are stored this way.

 ```php
$handler = new StackifyHandler('application_name', 'environment_name', $transport, true); 
```


### CurlTransport

CurlTransport does not require a Stackify agent to be installed and it also sends data directly to Stackify services. It collects log entries in a single batch and sends data using native [PHP cURL](http://php.net/manual/en/book.curl.php) functions. This way is a blocking one, so it should not be used on production environments. To configure CurlTransport you need to pass environment name and API key (license key):

PHP:
```php
use Stackify\Log\Transport\CurlTransport;
use Stackify\Log\Monolog\Handler as StackifyHandler;
    
$transport = new CurlTransport('api_key');
$handler = new StackifyHandler('application_name', 'environment_name', $transport);
$logger = new Logger('logger');
$logger->pushHandler($handler);
```

Symfony: 
```yml
services:
    stackify_transport:
        class: "Stackify\\Log\\Transport\CurlTransport"
        arguments: ["api_key"]
    stackify_handler:
        class: "Stackify\\Log\\Monolog\\Handler"
        arguments: ["application_name", "environment_name", "@stackify_transport"]
monolog:
    handlers:
        stackify:
            type: service
            id: stackify_handler
```

#### Optional Configuration

**Proxy**
- CurlTransport supports data delivery through proxy. Specify proxy using [libcurl format](http://curl.haxx.se/libcurl/c/CURLOPT_PROXY.html): <[protocol://][user:password@]proxyhost[:port]>
```php
$transport = new CurlTransport($apiKey, ['proxy' => 'https://55.88.22.11:3128']);
```

**Log Server Environment Variables**
- Server environment variables can be added to error log message metadata. **Note:** This will log all 
system environment variables; do not enable if sensitive information such as passwords or keys are stored this way.

 ```php
$handler = new StackifyHandler('application_name', 'environment_name', $transport, true); 
```

### AgentTransport

AgentTransport does not require additional configuration in your PHP code because all data is passed to the [Stackify agent](https://stackify.screenstepslive.com/s/3095/m/7787/l/119709-installation-for-linux). The agent must be installed on the same machine. Local TCP socket on port 10515 is used, so performance of your application is affected minimally.

PHP:
```php
use Monolog\Logger;
use Stackify\Log\Monolog\Handler as StackifyHandler;

$handler = new StackifyHandler('application_name', 'environment_name');
$logger = new Logger('logger');
$logger->pushHandler($handler);
```

Symfony: 
```yml
services:
    stackify_handler:
        class: "Stackify\\Log\\Monolog\\Handler"
        arguments: ["application_name", "environment_name"]
monolog:
    handlers:
        stackify:
            type: service
            id: stackify_handler
```

You will need to enable the TCP listener by checking the "PHP App Logs (Agent Log Collector)" in the server settings page in Stackify. See [Log Collectors Page](http://docs.stackify.com/m/7787/l/302705-log-collectors) for more details.

#### Optional Settings

**Log Server Environment Variables**
- Server environment variables can be added to error log message metadata. **Note:** This will log all 
system environment variables; do not enable if sensitive information such as passwords or keys are stored this way.

 ```php
$handler = new StackifyHandler('application_name', 'environment_name', null, true); 
```

## Notes

To get more error details pass Exception objects to the logger if available:
```php
try {
    $db->connect();
catch (DbException $ex) {
    // you may use any key name
    $logger->addError('DB is not available', ['ex' => $ex]);
}
```

## Troubleshooting
If transport does not work, try looking into ```vendor\stackify\logger\src\Stackify\debug\log.log``` file (if it is available for writing). Errors are also written to global PHP [error_log](http://php.net/manual/en/errorfunc.configuration.php#ini.error-log).
Note that ExecTransport does not produce any errors at all, but you can switch it to debug mode:
```php
$transport = new ExecTransport($apiKey, ['debug' => true]);
```

## License

Copyright 2018 Stackify, LLC.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
