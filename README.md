### Aqui está a criação da ativiade solicitada, com os arquivos YAML para cada etapa e os comandos necessários para aplicar e verificar os recursos no Kubernetes.



* ##### Crie um pod chamado "my-pod" usando uma imagem simples como "nginx" e verifique seu estado com os comandos de monitoramento do Kubernetes.

    ###### Arquivo YAML: my-pod.yaml
    ```bash
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
    spec:
      containers:
        - name: nginx-container
          image: nginx

###### Comandos
    kubectl apply -f my-pod.yaml
    kubectl get pods
    kubectl describe pod my-pod

* ##### Implante um Deployment chamado "my-deployment" com três réplicas de uma aplicação baseada na imagem "httpd". Atualize a imagem do Deployment para uma versão mais recente.

    ###### Arquivo YAML: my-deployment.yaml
    ```bash
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: my-app
      template:
        metadata:
          labels:
            app: my-app
        spec:
          containers:
            - name: httpd-container
              image: httpd:2.4

###### Comandos
    kubectl apply -f my-deployment.yaml
    kubectl get deployments
    kubectl get pods -l app=my-app
    kubectl set image deployment/my-deployment httpd-container=httpd:2.4.54
    kubectl rollout status deployment/my-deployment

* ##### Crie um ConfigMap chamado "app-config" com uma variável de configuração personalizada. Monte o ConfigMap em um pod e verifique se o valor foi aplicado corretamente.
    
    ###### Arquivo YAML: app-config.yaml
    ```bash
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config
    data:
      my-config-key: "custom-value"


###### Arquivo YAML: config-pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: config-pod
    spec:
      containers:
        - name: app-container
          image: nginx
          env:
            - name: MY_CONFIG
              valueFrom:
                 configMapKeyRef:
                   name: app-config
                   key: my-config-key

###### Comandos
    kubectl apply -f app-config.yaml
    kubectl apply -f config-pod.yaml
    kubectl exec config-pod -- printenv | grep MY_CONFIG

* ##### Crie um Secret chamado "app-secret" contendo informações sensíveis. Injete o Secret como uma variável de ambiente em um pod e teste se está acessível.

###### Arquivo YAML: app-secret.yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: app-secret
    type: Opaque
    data:
      my-secret-key: bXktc2VjcmV0LXZhbHVl

###### Arquivo YAML: secret-pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: secret-pod
    spec:
      containers:
        - name: app-container
          image: nginx
          env:
            - name: MY_SECRET
              valueFrom:
                secretKeyRef:
                name: app-secret
                key: my-secret-key

###### Comandos
    kubectl apply -f app-secret.yaml
    kubectl apply -f secret-pod.yaml
    kubectl exec secret-pod -- printenv | grep MY_SECRET

* ##### Configure um PersistentVolume de 1Gi de armazenamento local e vincule-o a um PersistentVolumeClaim. Monte o volume em um pod e salve arquivos para verificar a persistência.

###### Arquivo YAML: pv.yaml

        apiVersion: v1
        kind: PersistentVolume
        metadata:
        name: local-pv
        spec:
          capacity:
            storage: 1Gi
          accessModes:
            - ReadWriteOnce
          hostPath:
            path: /mnt/data

###### Arquivo YAML: volume-pod.yaml

    apiVersion: v1
    kind: Pod
    metadata:
    name: volume-pod
    spec:
      containers:
        - name: app-container
          image: nginx
          volumeMounts:
            - mountPath: /data
            name: storage
      volumes:
        - name: storage
          persistentVolumeClaim:
            claimName: local-pvc

###### Comandos

    kubectl apply -f pv.yaml
    kubectl apply -f pvc.yaml
    ubectl apply -f volume-pod.yaml
    kubectl exec volume-pod -- touch /data/test-file

* ##### Crie um serviço do tipo ClusterIP para um Deployment chamado "backend" e teste a conectividade interna entre pods usando o nome do serviço.

###### Arquivo YAML: backend-deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: backend
    spec:
      replicas: 3
      selector:
        matchLabels:
        app: backend
      template:
        metadata:
          labels:
            app: backend
        spec:
          containers:
            - name: backend-container
              image: nginx

###### Arquivo YAML: backend-service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: backend-service
    spec:
      selector:
        app: backend
      ports:
        - protocol: TCP
        port: 80
        targetPort: 80
    type: ClusterIP

###### Comandos:

    kubectl apply -f backend-deployment.yaml
    kubectl apply -f backend-service.yaml
    kubectl exec -it backend-pod -- curl backend-service

* ##### Implante um Job chamado "batch-job" que execute um comando simples e termine. Verifique os logs do Job para confirmar sua execução.

###### Arquivo YAML: batch-job.yaml

    apiVersion: batch/v1
    kind: Job
    metadata:
      name: batch-job
    spec:
      template:
        spec:
          containers:
            - name: batch-container
              image: busybox
              command: ["echo", "Hello from Batch Job!"]
          restartPolicy: OnFailure

###### Comandos:
    
    kubectl apply -f batch-job.yaml
    kubectl logs job/batch-job

###### Crie um Horizontal Pod Autoscaler para um Deployment chamado "hpa-deployment" e configure-o para escalar com base no uso de CPU. Aumente a carga e observe o escalonamento.

###### Arquivo YAML: hpa-deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: hpa-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
        app: hpa-app
      template:
        metadata:
          labels:
            app: hpa-app
        spec:
          containers:
          - name: hpa-container
            image: nginx
            resources:
              requests:
                cpu: "100m"
              limits:
                cpu: "200m"

###### Comandos:
    kubectl apply -f hpa-deployment.yaml

###### Crie o HPA configurado para escalar com base no uso de CPU. Use o seguinte comando:
    kubectl autoscale deployment hpa-deployment --cpu-percent=50 --min=1 --max=5

###### Esse comando configura o HPA para:
 * ###### Escalar quando o uso médio de CPU dos pods ultrapassar 50%.
 * ###### Ter no mínimo 1 pod e no máximo 5 pods.

###### Verifique se o HPA foi criado:
    kubectl get hpa

###### Gerar Carga para Testar o Escalonamento

    kubectl run load-generator --image=busybox --restart=Never -- \
    sh -c "while true; do wget -q -O- http://hpa-deployment-service:80; done"

###### Observar o Escalonamento
    kubectl get hpa -w

###### Verifique o número de réplicas atual:
    kubectl get deployment hpa-deployment

* ##### Crie um serviço do tipo NodePort para expor externamente um Deployment chamado "webapp". Acesse o serviço usando o endereço IP do Minikube e a porta atribuída

###### Arquivo YAML: webapp-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: webapp
    spec:
      replicas: 2
      selector:
        matchLabels:
        app: webapp
      template:
        metadata:
          labels:
            app: webapp
        spec:
          containers:
            - name: web-container
              image: nginx


###### Arquivo YAML: webapp-service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: webapp-service
    spec:
      selector:
        app: webapp
      ports:
        - protocol: TCP
          port: 80
          nodePort: 30007
      type: NodePort

###### Comandos:
    kubectl apply -f webapp-deployment.yaml
    kubectl apply -f webapp-service.yaml
    minikube service webapp-service

* ##### Crie um pod chamado "restart-pod" com a política de reinício configurada como "OnFailure". Provoque uma falha no pod e observe seu comportamento.

###### Arquivo YAML: restart-pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: restart-pod
    spec:
      containers:
        - name: fail-container
          image: busybox
          command: ["sh", "-c", "exit 1"]
      restartPolicy: OnFailure

###### Comandos:
    kubectl apply -f restart-pod.yaml
    kubectl describe pod restart-pod

