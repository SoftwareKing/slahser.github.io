随着开发的推进很快团队就有了内网服务调用的需求. 


```
docker run -d -p 443:5000 --restart=always --name registry \
  -v /.ssh:/certs \
  -v /opt/docker-image:/opt/docker-image \
  -e STORAGE_PATH=/opt/docker-image \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.gogen.work.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/registry.gogen.work.key \
  registry:2
```