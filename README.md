# snoke/ws-bundle

Symfony bundle for the Snoke WebSocket Gateway stack.

This repository contains **only** the Symfony integration (publishers, presence, inbox consumer, token helper).
To run a full local stack (gateway + brokers), use the gateway repo:
`https://github.com/snoke/Symfony-Python-Realtime-Stack`

## Requirements
- PHP >= 8.4
- Symfony ^8.0

## Install
```
composer require snoke/ws-bundle:0.1.2
```
If you want the latest changes, use `dev-main`.

Note: the bundle depends on `open-telemetry/transport-grpc`, which requires the PHP `grpc` extension.
For local dev you can either install `ext-grpc` or use:
```
composer install --ignore-platform-req=ext-grpc
```

If you are not using Symfony Flex, ensure the bundle is registered in `config/bundles.php`:
```
return [
  Snoke\WsBundle\SnokeWsBundle::class => ['all' => true],
];
```

## Configuration
Create `config/packages/snoke_ws.yaml` in your Symfony app:
```
snoke_ws:
  mode: '%env(default:ws_mode_default:WS_MODE)%'
  transport:
    type: '%env(default:ws_transport_type_default:WS_TRANSPORT_TYPE)%'
    http:
      base_url: '%env(resolve:default::WS_GATEWAY_BASE_URL)%'
      publish_path: '/internal/publish'
      timeout_seconds: 5
      auth:
        type: api_key
        header: 'X-Api-Key'
        value: '%env(resolve:default::WS_GATEWAY_API_KEY)%'
    redis_stream:
      dsn: '%env(resolve:default::WS_REDIS_DSN)%'
      stream: '%env(resolve:default::WS_REDIS_STREAM)%'
  presence:
    type: '%env(default:ws_presence_type_default:WS_PRESENCE_TYPE)%'
    http:
      base_url: '%env(resolve:default::WS_GATEWAY_BASE_URL)%'
      list_path: '/internal/connections'
    redis:
      dsn: '%env(resolve:default::WS_REDIS_DSN)%'
      prefix: '%env(resolve:default::WS_REDIS_PREFIX)%'
  events:
    type: '%env(default:ws_events_type_default:WS_EVENTS_TYPE)%'
    redis_stream:
      dsn: '%env(resolve:default::WS_REDIS_DSN)%'
      stream: '%env(resolve:default::WS_REDIS_EVENTS_STREAM)%'
  subjects:
    user_prefix: 'user:'
```

Defaults:
- `terminator` mode uses HTTP publish + webhook events
- `core` mode uses Redis streams

## Core mode consumer
In core mode you must run a consumer that reads `ws.inbox` and dispatches events:
```
php bin/console ws:consume
```

You can run this as a separate service in Docker Compose.

## Demo token helper (optional)
The bundle includes a demo token service for quick local testing:
```
use Snoke\WsBundle\Service\DemoTokenService;

public function token(DemoTokenService $tokens): Response
{
    [$jwt, $error] = $tokens->issue('42');
    // return token to the frontend
}
```

## Gateway stack
To start the gateway + brokers stack, use the gateway repo:
`https://github.com/snoke/Symfony-Python-Realtime-Stack`
