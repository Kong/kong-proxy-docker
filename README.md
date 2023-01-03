# kong-proxy-docker

## How to run
Configure the license env variable
```shell
docker-compose up
```
Add entry for karaboondi.com to 127.0.0.1 in /etc/hosts

Run the curl command to check the headers being passed

```shell
cd tls
curl --cert ./client.cert-chain.pem --key ./client.key.pem --cacert ./ca.cert.pem https://karaboondi.com:8443/headers
```
