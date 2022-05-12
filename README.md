# Check out the repository with submodules
git clone --recurse-submodules https://github.com/pawel-kow/DomainConnectDNSProviderDemo.git
# Starting
docker-compose up -d
# Starting for development
docker-compose up -f docker-compose-dev.yml -d

# First run
Login over http://localhost:9191, register as a new user (will become admin automatically)
Configure PDNS connection:
URL: http://pdns:8081/
Key: same as PDNS_AUTH_API_KEY in docker-compose.yml file
Version: 4.6

Create a zone foo.com

# Logs

# Test PDNS
## Dig installed locally:
dig ANY @localhost -p 1053 foo.com
## with Docker
docker run -it tutum/dnsutils dig ANY @host.docker.internal -p 1053 foo.com

# Working with templates
Templates can be modified any time in ./Templates folder.
The folder is mounted in docker and the changes are visible immediately.
