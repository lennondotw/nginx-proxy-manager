# Public HTTP and HTTPS ports

This fork is based on Nginx Proxy Manager 2.16.0. Its image builds the complete fork backend, frontend and startup scripts on the pinned official runtime, including the public-port contribution on current upstream develop. The first image supports `linux/amd64`.

```yaml
services:
  app:
    image: ghcr.io/lennondotw/nginx-proxy-manager:2.16.0-public-ports
    environment:
      PUBLIC_HTTP_PORT: "232"
      PUBLIC_HTTPS_PORT: "233"
    ports:
      - "232:80"
      - "233:443"
      - "1906:81"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    restart: unless-stopped
```

`PUBLIC_HTTP_PORT` defaults to 80 and `PUBLIC_HTTPS_PORT` defaults to 443. Values must be decimal integers between 1 and 65535. They describe the ports clients access; configure Docker/router mappings to match. Recreate the container after changing them. The same image supports all valid configurations.

Force SSL redirects retain the upstream 301 status, path, query, ACME test exception, and trusted-forwarded-protocol behavior. For a nonstandard HTTPS port, HTTP sent to an SSL listener (Nginx 497) redirects with 307 to that port. Standard 443 retains the upstream 497 behavior.

The manager opens certificate-enabled hosts using HTTPS, and other hosts using HTTP. Both the visible domain label and link include a nonstandard port. This applies to Proxy, Redirection, and 404 Hosts, independently of the upstream forwarding protocol. Wildcard links remain non-navigable. Certificate-list links keep their original behavior.

Explicit redirection destinations, the default-site URL, upstream ports, upstream Host headers, and application-generated absolute URLs are managed separately. Port configuration is not stored in the database and has no settings form. The existing health response exposes `public_ports` to the manager.

When migrating this homelab, back up and remove the old `error_page 497 301 =307 ...:233...` rule in `/data/nginx/custom/server_proxy.conf`; otherwise it will continue to override redirects. The previous rule made Force SSL return 307, while this fork retains the official 301. Keep other service-specific advanced settings.

Build the manager with `yarn --cwd frontend install --frozen-lockfile`, `yarn --cwd frontend locale-compile`, and `yarn --cwd frontend build`. Then build `docker/Dockerfile.public-ports` with `--platform linux/amd64`. The base image is pinned by digest; the complete backend, its frozen-lockfile production dependencies, rebuilt frontend and startup configuration are installed from this checkout. Public-port configuration runs after file-based environment variables are loaded.
