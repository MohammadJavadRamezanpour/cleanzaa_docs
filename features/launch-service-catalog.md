# Active Ferrara service catalog (B08)

## Purpose and scope

`GET /catalog/launch` publishes the current read-only pilot catalog. Version `ferrara-v1` contains one area, `IT-FE-FERRARA`, and one service, `HOME_CLEANING`. The city is Ferrara, country `IT`, timezone `Europe/Rome`, and coarse inclusive CAP range `44121`–`44124`. The previous `milano-v1` catalog with three services is historical and is not served as the active launch catalog. [ADR 0010](../adr/0010-ferrara-home-cleaning-pilot.md) records the change.

The response still includes the shared date, time, address, recipient, equipment/supplies, and whole-hour duration fields. Duration is 1–8 whole hours. Home Cleaning requires the room/parts description, positive room count, and beginner/professional preference. The worker must declare a level and obtain manual review. Car Wash and Pet Care are not active and cannot be newly quoted or saved as a draft request. The CAP gate is a coarse input check, not proof of an exact street's municipal location.

## API contract

`GET /catalog/launch` is public, with no path, query, header, or body parameters, pagination, ordering, or authentication. It returns `200 LaunchCatalog`, `Cache-Control: public, max-age=300`, and `X-Request-ID`. Its area carries code/city/country/timezone/CAP bounds; its single service carries code/name/summary, required booking fields, qualification summary, `home_level_options`, certificate and manual-review flags. Infrastructure failures use the shared error envelope. The backend's active `is_supported_address` and `is_supported_service` functions reject Milano, other cities/CAPs, Car Wash, and Pet Care. The server repeats these checks for draft quotes and new draft requests.

This response changes the active version and removes two service choices from the previous API. It is a client compatibility change: the frontend pins the new OpenAPI export and must deploy with the backend. Historical `milano-v1` values remain in stored snapshots for reading, not for new submission. A future new market requires another version and migration rather than relabeling old records.

## Browser behavior and verification

`/services` renders the one service and Ferrara CAP range from the API. `/book` uses the same catalog. The UI shows that the example is a synthetic demo and does not promise a booking. Backend integration tests assert the current catalog, address boundary, inactive-service rejection, and historical-version retention. Run `PYTHONPATH=. pytest -q`, `ruff check .`, `ruff format --check .`, `mypy app scripts`, `alembic check`, and `python -m scripts.export_openapi --check` with the backend environment. Frontend checks are `npm test`, `npm run lint`, `npm run format:check`, `npm run typecheck`, `npm run contract:check`, and `npm run build`.
