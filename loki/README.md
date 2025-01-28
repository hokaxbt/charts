# Loki Charts

## Prerequisites

1. Create a namespace:

    ```sh
    kubectl create ns grafana
    ```

2. Create a secret for Loki S3:

    ```sh
    kubectl create secret generic hetzner-s3-credentials -n grafana \
        --from-literal=S3_ENDPOINT='XXXX' \
        --from-literal=S3_REGION='XXXX' \
        --from-literal=S3_ACCESS_KEY='XXXX' \
        --from-literal=S3_SECRET_KEY='XXXX'
    ```

## Deployment

Deploy Loki using the following command:

```sh
helm install loki ./loki \
    --set volume.storageClass="openebs-lvm" \
    --set s3.buckets.chunks="kubernetes-loki-chunks" \
    --set s3.buckets.rulers="kubernetes-loki-rulers" \
    --set secret="hetzner-s3-credentials" \
    -n grafana
```

Dry run:

```sh
helm install loki ./loki \
    --set volume.storageClass="openebs-lvm" \
    --set s3.buckets.rulers="test-rulers-bucket" \
    --set s3.buckets.chunks="test-chunks-bucket" \
    --set secret="hetzner-s3-credentials" \
    -n grafana \
    --dry-run
```

## Verification

Check the deployments to ensure they are healthy and distributed across 3 nodes:

```sh
kubectl get pods -n grafana -o wide
```

## Sending Logs

1. Port-forward the Loki service:

    ```sh
    kubectl port-forward -n grafana svc/loki 3100:80
    ```

2. Send logs using the following command:

    ```sh
    curl -H "Content-Type: application/json" \
        -XPOST -s "http://127.0.0.1:3100/loki/api/v1/push"  \
        --data-raw "{\"streams\": [{\"stream\": {\"job\": \"test\"}, \"values\": [[\"$(date +%s)000000000\", \"fizzbuzz\"]]}]}"
    ```

3. Verify that Loki received the data:

    ```sh
    curl "http://127.0.0.1:3100/loki/api/v1/query_range" --data-urlencode 'query={job="test"}' | jq .data.result
    ```

## References

- [Grafana Loki configuration parameters](https://grafana.com/docs/loki/latest/configure/)