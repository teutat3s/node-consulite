# consulite
Tiny consul Node.js module for client discovery with round-robin support

[![Npm Version](https://img.shields.io/npm/v/consulite.svg)](https://npmjs.com/package/consulite)
[![Node Version](https://img.shields.io/node/v/consulite.svg)](https://npmjs.com/package/consulite)
[![Build Status](https://secure.travis-ci.org/joyent/node-consulite.svg)](http://travis-ci.org/joyent/node-consulite)

## API


### new Consulite(options)

Create a new instance of the Consulite class. `options` can include the following properties:

* `consul` - consul host to connect to. Defaults to either:
  * `${process.env.CONSUL_HOST}:${process.env.CONSUL_PORT}`
  * `consul:8500` - as a last resort


### getServiceNames([, callback])

Get all service names from consul, regardless of health status.

* `callback`: function with the signature `(err, services)` where `services` is an array of service names.

This function returns a `Promise` if no `callback` is provided.


### getServiceStatus(name, [, callback])

Get all nodes for the service and include their health status.

* `callback`: function with the signature `(err, nodes)` where `nodes` is an array of node data, which is formatted with the properties shown in the example below.

```
[
  {
    node: 'foobar',
    address: '10.1.10.12',
    port: 8000,
    status: 'passing'
  }
]
```

This function returns a `Promise` if no `callback` is provided.


### getService(name [, callback])

Get service address information from cache or from consul. When multiple service
instances are registered with consul the first instance that hasn't been executed
or the oldest executed service is returned.

* `name`: the service name registered with consul. If no services are found
then an error will be returned to the callback. If multiple services are found
then the service that hasn't been executed or hasn't been executed most recently
will be returned in the callback.

* `callback`: function with the signature `(err, service)` where `service` has
the following properties:
  - `address`: the host address where the service is located
  - `port`: the port that the service is exposed on

This function returns a `Promise` if no `callback` is provided.


### getServiceHosts(name [, callback])

Get an array of all service instances from cache or from consul.

* `name`: the service name registered with consul. If no services are found
then an error will be returned to the callback.

* `callback`: function with the signature `(err, hosts)` where `hosts` is an
array of objects with the following properties:
  - `address`: the host address where the service is located
  - `port`: the port that the service is exposed on

This function returns a `Promise` if no `callback` is provided.


### getCachedService(name)

Get the next service from the cache if it exists, otherwise return null;


### getCachedServiceHosts(name)

Get an array of all service instances from the cache if it exists, otherwise
null.


### refreshService(name [, callback])

Makes a request to consul for the given service name and caches the results. Only
services that are healthy are cached.

* `name`: the service name to fetch from consul.
* `callback`: function with signature `(err, services)`

This function returns a `Promise` if no `callback` is provided.


## Example Usage

```js
const Consulite = require('consulite');
const Wreck = require('wreck');

const consulite = new Consulite();
consulite.getService('users', (err, service) {
  if (err) {
    console.error(err);
    return;
  }

  Wreck.get(`http://${service.address}:${service.port}/users`, (err, res, payload) => {
    // handle error and do something with results
  });
});
```

If you want to set the consul address to use and don't want to depend on
environment variables you can use the `config()` function as demonstrated below:

```js
const Consulite = require('consulite');
const Wreck = require('wreck');

const consulite = new Consulite({ consul: 'http://myconsul.com' });

consulite.getService('users', (err, service) {
  if (err) {
    console.error(err);
    return;
  }

  Wreck.get(`http://${service.address}:${service.port}/users`, (err, res, payload) => {
    // handle error and do something with results
  });
});
```
