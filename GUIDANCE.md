# Nginx for Matomo on Wodby

What this service adds to the Nginx service it is based on.

## Preset

The service sets `NGINX_VHOST_PRESET` to `matomo`, the image's rule set for Matomo. Its template is declared as a config file of this service and can be overridden there. What it does:

- Only these scripts run as PHP: `index.php`, `matomo.php`, `piwik.php`, `js/index.php` and `plugins/HeatmapSessionRecording/configs.php`. Every other `.php` path returns 403.
- The directories `config`, `tmp`, `core` and `lang` are denied.
- Under `libs`, `vendor`, `plugins`, `misc` and `node_modules`, only files with a static extension are served; everything else there is denied. The static extensions of this preset include `json` and `html`.
- Tag Manager preview containers (`js/container_*_preview.js`) are sent with caching disabled.
- A path that matches no file returns 404. There is no fallback to `index.php`.

## Backend link and build

- The required backend link must point to the Matomo PHP service. It sets `NGINX_BACKEND_HOST` and `NGINX_BACKEND_PORT`, which form the FastCGI upstream.
- This service has no repository of its own: its image is built from the source of the linked Matomo service, copied to `/var/www/html`. Its `.dockerignore` leaves out `vendor`, `node_modules`, `core`, `tests`, `console`, the Composer and npm manifests and dot files, so those are not in the web server image.
- The `docroot` setting of the Nginx service applies; leave it empty for a Matomo codebase at the repository root.

A file that Matomo writes at run time in the PHP service's container, for example a plugin installed from the interface, is not in this service's image.

## Check the result

- `nginx -T` shows the preset and upstream in effect.
- `curl -sI localhost/matomo.php` from inside the container returns Matomo's response when the PHP service is reachable.
