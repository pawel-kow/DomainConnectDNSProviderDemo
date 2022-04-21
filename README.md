# Starting
docker-compose up -d
# First run
Login over http://localhost:9191, create a new admin user
Create a zone
# Test PDNS
## Dig installed locally:
docker run -it tutum/dnsutils dig ANY @host.docker.internal -p 1053 foo.com
## with Docker
dig ANY @localhost -p 1053 foo.com
