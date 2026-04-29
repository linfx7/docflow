## Feature IDs

`YYMMDD-name` format. Date prefix is today's date (creation date). Name is a slug: lowercase letters, digits, hyphens only. Example: `260430-user-login`.

## MD5 hash

Used for drift detection in Associated Files. Compute with: `md5 -q <file>` (macOS), fallback `md5sum <file> | cut -d' ' -f1` (Linux).
