Symfony Twirp Handler
=====================


Helps implementing Twirp in a Symfony application. 


Lets say you have this service defined in a proto file:

```
option php_generic_services = true;

service SearchService {

    rpc Search (SearchRequest) returns (SearchResponse);

}
```

From this file, protoc generates a generic service interface 
`SearchServiceInterface.php`. You just implement this interface with your 
business logic. 

Then you can let `TwirpHandler` take care of request and response:  


```php
/**
 * @Route( path="twirp/{serviceName}/{methodName}" )
 */
public function execute(RequestInterface $request, string $serviceName, string $methodName): Response
{

    $resolver = new ServiceResolver();
    $resolver->registerInstance(
        SearchServiceInterface::class, // the interface generated by protoc 
        new SearchService() // your implementation of the interface
    );

    $handler = new TwirpHandler($resolver);

    return $handler->handle($serviceName, $methodName, $request);
}
```

Twirp has its own error format. To convert exceptions (by the security 
firewall or from within your service implementation), you can use the 
`TwirpErrorSubscriber`: 

```yaml
// services.yaml
SymfonyTwirp\TwirpErrorSubscriber:
    arguments:
        $requestTagAttribute: "_request_id"
        $debug: '%kernel.debug%'
        $prefix: "api"twirp
```

For documentation about the arguments, check the PHPdoc of `TwirpErrorSubscriber`.
