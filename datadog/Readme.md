## Temporal -> Prometheus -> Metrics Datadog

This is a demo of getting metrics from Temporal over to datadog using prometheus. 


## Step 1. Finish steps in Root folder, ensure docker-compose is up
(brew install grpcurl)
```
docker ps 

grpcurl -plaintext -d '{"service": "temporal.api.workflowservice.v1.WorkflowService"}' 127.0.0.1:7233 grpc.health.v1.Health/Check

```

## Step 2. Get the network ID 

In the parent directions, you ran `docker network create temporal-network`.  We need to get the network id for this network. 

In this case the id will be 17d3ef8f14fb
```
# docker network ls | grep temp
    
    17d3ef8f14fb  temporal-network  bridge  local

#
```


## Step 3. Run the docker container to run datadog. 
```
docker run -d  --cgroupns host  \
    --pid host \
    -v [FULL_PATH]/conf.yaml:/etc/datadog-agent/conf.d/prometheus.d/conf.yaml:ro \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -v /proc/:/host/proc/:ro \
    -p 8125:8125 -p 8126:8126 \
    -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
    -e DD_API_KEY="[DD_API_KEY]" \
    -e DD_APM_NON_LOCAL_TRAFFIC=true \
    --network="[NETWORK_ID]" \
    gcr.io/datadoghq/agent:latest
```


### Example: 
```
docker run -d  --cgroupns host  \
    --pid host \
    -v /home/user/conf.yaml:/etc/datadog-agent/conf.d/prometheus.d/conf.yaml:ro \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -v /proc/:/host/proc/:ro \
    -p 8125:8125 -p 8126:8126 \
    -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
    -e DD_API_KEY="[DD_API_KEY]" \
    -e DD_APM_NON_LOCAL_TRAFFIC=true \
    --network="17d3ef8f14fb" \
    gcr.io/datadoghq/agent:latest
```
