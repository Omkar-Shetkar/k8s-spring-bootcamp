
### Create Spring boot project

`$ curl https://start.spring.io/starter.tgz -d artifactId=k8s-demo-app -d name=k8s-demo-app -d packageName=com.example.demo -d dependencies=web,actuator -d javaVersion=11 | tar -xzf -
`
### Run the project

`$ ./mvnw spring-boot:run`

### Build image of Spring boot project

`$ ./mvnw spring-boot:build-image`

### Run the container

`$ docker run --name k8s-demo-app -p 8080:8080 k8s-demo-app:0.0.1-SNAPSHOT`

### Start local docker registory (optional) else docker hub can be used

`docker run -p 5000:5000 registry`

### Build and push to local docker registry

`$ ./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=localhost:5000/apps/demo`

`$ docker push localhost:5000/apps/demo`

### Check the image in registry

`$ curl localhost:5000/v2/_catalog`

### Kubernetes deployment

`$ kubectl create deployment k8s-demo-app --image localhost:5000/apps/demo -o yaml --dry-run=client > k8s/deployment.yaml`

`$ kubectl create service clusterip k8s-demo-app --tcp 80:8080 -o yaml --dry-run=client > k8s/service.yaml`

`$ kubectl apply -f ./k8s`

`$ kubectl port-forward service/k8s-demo-app 8080:80`

`$ curl http://localhost:8080; echo`

### Converting the service to LoadBalancer type

`$ kubectl patch service k8s-demo-app -p '{"spec": {"type": "LoadBalancer", "externalIPs":["172.18.0.2"]}}'`

### Deploy with Skaffold

For k3s/microk8s

`$ kubectl config view --raw > $HOME/.kube/config`

`$ skaffold dev --port-forward`

`$ skaffold debug --port-forward`

### Using Kustomize

`$ mkdir -p kustomize/base
 $ mv k8s/* kustomize/base
 $ rm -Rf k8s`

`$ kustomize build ./kustomize/base`

`$ kustomize build ./kustomize/qa`

`$ kustomize build kustomize/qa | kubectl apply -f -`
`$ kustomize build kustomize/qa | kubectl delete -f -`

### Integration with Skaffold
```yaml
deploy:
  kustomize:
    paths: ["kustomize/base"]
profiles:
  - name: qa
    deploy:
      kustomize:
        paths: ["kustomize/qa"]
```

`$ skaffold dev --port-forward`
`$ skaffold dev -p qa --port-forward`

### Externalized Configuration

`$ kubectl create configmap log-level --from-literal=LOGGING_LEVEL_ORG_SPRINGFRAMEWORK=DEBUG`

