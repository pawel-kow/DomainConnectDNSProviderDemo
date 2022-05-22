# Check out the repository with submodules
git clone --recurse-submodules https://github.com/pawel-kow/DomainConnectDNSProviderDemo.git
# Starting for development

## Configure ngrok
- Open a free account at ngrok.com
- Get a key file
- change `NGROK_AUTHTOKEN` env variable in `docker-compose-dev.yml` or create `.env` with this variable configuration

## Start a local instance with mounted source code and live changes
`docker-compose up -f docker-compose-dev.yml -d`
# First run
Login over http://localhost:9191, register as a new user (will become admin automatically)
Configure PDNS connection:
- URL: `http://pdns:8081/`
- Key: same as PDNS_AUTH_API_KEY in docker-compose.yml file
- Version: 4.6

Create a zone foo.com

## Add `_domainconnect` record to your zone
- Check in ngrok control panel what subdomain the service is running at.

Note: For free account at ngrok the host will change each time the agent is restarted

- Add `_domainconnect` TXT record to your domain name foo.com
  - TTL: as low as possible in order to change the host quickly if needed
  - value: `{ngrok subdomain}/dc`
# Querying local PDNS do DNS records
## Dig installed locally:
`dig ANY @localhost -p 1053 foo.com`
## with Docker
`docker run -it tutum/dnsutils dig ANY @host.docker.internal -p 1053 foo.com`

# Working with templates
Templates can be modified any time in ./Templates folder.
The folder is mounted in docker and the changes are visible immediately.

# Starting a standalone public instance
`docker-compose up -d`

Note: in order for the nameservers to work correctly the machine requires 2 IPs configured

## Add firewall rules for DNS
Open port 53 TCP&UDP

## Add https gateway
Example nginx configuration with LetsEncrypt:
```
server {
    listen 443 ssl;
    server_name {YOUR_DOMAIN_NAME};
    server_tokens off;

    ssl_certificate /etc/letsencrypt/live/{YOUR_DOMAIN_NAME}/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/{YOUR_DOMAIN_NAME}/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    client_max_body_size 64M;

    location / {
        proxy_pass          http://{local docker interface IP}:9191/;
        proxy_set_header    Host               $host;
        proxy_set_header    X-Real-IP          $remote_addr;
        proxy_set_header    X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Host   $host;
        proxy_set_header    X-Forwarded-Server $host;
        proxy_set_header    X-Forwarded-Port   $server_port;
        proxy_set_header    X-Forwarded-Proto  $scheme;
    }
}
```