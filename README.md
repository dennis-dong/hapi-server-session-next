# hapi-server-session-next

Simple server-side session support for hapi

[![npm version](https://badge.fury.io/js/hapi-server-session-next.svg)](https://badge.fury.io/js/hapi-server-session-next)

## Install

    $ npm install hapi-server-session-next
## Example

```javascript
'use strict';

const hapi = require('@hapi/hapi');

const main = async () => {
  const server = new hapi.Server({
    host: 'localhost',
    address: '127.0.0.1',
    port: 8000,
  });

  await server.register({
    plugin: require('hapi-server-session-next'),
    options: {
      cookie: {
        isSecure: false, // never set to false in production
      },
    },
  });

  server.route({
    method: 'GET',
    path: '/',
    handler: (request, _h) => {
      request.session.views = request.session.views + 1 || 1;
      return 'Views: ' + request.session.views;
    },
  });

  await server.start();
};

main().catch(console.error);
```

## Options

- `algorithm`: [Default: `'sha256'`] algorithm to use during signing
- `cache`: supports the same options as [`server.cache(options)`](<https://hapijs.com/api#server.cache()>)
  - `expiresIn`: [Default: session `expiresIn` if set or `2147483647`] session cache expiration in milliseconds
  - `segment`: [Default: `'session'`] session cache segment
- `cookie`: supports the same options as [`server.state(name, [options])`](<https://hapijs.com/api#server.state()>)
  - `isSameSite`: [Default: `'Lax'`] sets the `SameSite` flag
  - `path`: [Default: `'/'`] sets the `Path` flag
  - `ttl`: [Default: session `expiresIn` if set] sets the `Expires` and `Max-Age` flags
- `expiresIn`: session expiration in milliseconds
- `name`: [Default: `'id'`] name of the cookie
- `key`: signing key. Prevents weaknesses in randomness from affecting overall security
- `size`: [Default: `16`] number of random bytes in the session id
- `vhost`: [Default: `*`] host or list of hosts that plugin should be enabled for

## Questions

### Can you explain what the `expiresIn` and `ttl` options do?

When the session `expiresIn` is not set, the cookie `ttl` is not set and the cache `expiresIn` is `2147483647`. This creates a true session cookie, i.e. one that is deleted when the browser is closed, but will last forever otherwise. This is the default with no configuration.

When the session `expiresIn` is set, it defaults both the cookie `ttl` and the cache `expiresIn` to the same value. This creates a session that will last `expiresIn` milliseconds. Even if the cookie `ttl` is ignored by the browser, the server-side cache will expire.

More complex configurations are possible. For example, when the session `expiresIn` is set and the cookie `ttl` is explicitly set to `null`, a session will last until the browser is closed, but no longer than `expiresIn` milliseconds.

### How do I destroy the session (e.g. to logout a user)?

```javascript
delete request.session;
```

will unset the cookie and delete the session from the cache.

## Changes

### [v6.0.1](https://github.com/dennis-dong/hapi-server-session-next/compare/v6.0.1...v6.0.1)

- support hapi v21
