# Laravel storage on the hosting server

The file session driver writes to `storage/framework/sessions`. If that directory
is absent, requests fail with `file_put_contents(...): No such file or directory`.
Previously, the repository ignored all of `storage`, so Git deployments omitted
the required directories. The directory `.gitignore` files now preserve the
structure while excluding sessions, logs, caches, and uploaded files.

## Repair the existing deployment

In cPanel Terminal or SSH, run these commands as the hosting account that owns
the application. Use the PHP 8.2 CLI supplied by the host if `php` points to a
different version.

```sh
cd /home3/satellit/spacelinkkenya.co.ke || exit 1
mkdir -p storage/app/public storage/framework/cache/data storage/framework/sessions storage/framework/testing storage/framework/views storage/logs bootstrap/cache
chmod 755 storage storage/app storage/app/public storage/framework storage/framework/cache storage/framework/cache/data storage/framework/sessions storage/framework/testing storage/framework/views storage/logs bootstrap/cache
php artisan config:clear
php artisan view:clear
```

PHP must run as the directory owner for these `755` permissions to allow writes.
If PHP uses a different account, have the host set the correct ownership or a
shared writable group for `storage` and `bootstrap/cache`.

For this public deployment, set these values in the server's `.env`:

```dotenv
APP_ENV=production
APP_DEBUG=false
```

Run `php artisan config:cache` on the server after updating `.env`. This rebuilds
configuration using the server's paths. Do not upload a locally generated
`bootstrap/cache/config.php` when moving between machines or directories.

Reload the page to verify the session error is resolved. If another error occurs,
inspect `storage/logs/laravel.log` while keeping public debug output disabled.

## Future deployments

Include the directory `.gitignore` files when deploying or uploading an archive;
tools that omit dotfiles can drop otherwise empty directories. Ensure the same
directories exist and are writable before running Artisan commands. Keep the
server's existing uploads and runtime data when updating the application.
