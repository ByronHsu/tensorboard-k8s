## Build tensorboard via Kubernetes

1. `minikube start --kubernetes-version v1.14.3`
2. `helm install submarine ./helm-charts/submarine`
3. `kubectl apply -f tf-board-mnist.yaml`

## Connect to tensorboard

1. By `minikube service list`, I found that the traefik entry url is `172.17.0.2:32080`.
2. As described in the yaml file, the tensorboard service should be accessed by `172.17.0.2:32080/testpath`.

   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
   name: mnist-tfboard
   namespace: default
   labels:
     app: mnist-tfboard
   annotations:
     kubernetes.io/ingress.class: traefik
     traefik.ingress.kubernetes.io/rewrite-target: /
   spec:
   rules:
     - http:
         paths:
           - path: /testpath
         backend:
           serviceName: mnist-tfboard
           servicePort: 8080
   ```

3. However, I can not access the tensorboard from `172.17.0.2:32080/testpath`. When I check the log of the pod, it shows that:
   ```
   TensorBoard 1.11.0 at http://mnist-tfboard-79bf6b8bc-jkh5l:6006 (Press CTRL+C to quit)
   W1217 07:37:38.816917 Thread-1 application.py:300] path /testpath not found, sending 404
   W1217 07:37:38.816916 139931846440704 application.py:300] path /testpath not found, sending 40
   ```
   I suggested that it is because the request is forwarded from `172.17.0.2:32080/testpath` to `mnist-tfboard:8080/testpath` rather than `mnist-tfboard:8080`. But I have already set `traefik.ingress.kubernetes.io/rewrite-target: /`, it should not have the suffix of `/testpath`.
   Did anyone know the cause? Thanks.
