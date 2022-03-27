# Apuntes

https://www.youtube.com/watch?v=DCoBcpOA7W4

https://github.com/pablokbs/peladonerd/tree/master/kubernetes/35


https://www.youtube.com/watch?v=CV_Uf3Dq-EU&t=0s


## NameSpace    ( División lógica del clúster )

    $ kubectl get ns
    NAME              STATUS   AGE
    default           Active   3d12h
    kube-node-lease   Active   3d12h
    kube-public       Active   3d12h
    kube-system       Active   3d12h

## Pods ( Unidad mínima de kubernetes. Contiene uno o varios contenedores, preferiblemente uno.)

    $ kubectl -n kube-system get pods -o wide
    NAME                                     READY   STATUS    RESTARTS   AGE     IP             NODE             NOMINATED NODE   READINESS GATES
    coredns-558bd4d5db-4q6lx                 1/1     Running   12         3d12h   10.1.0.52      docker-desktop   <none>           <none>
    coredns-558bd4d5db-dmn22                 1/1     Running   12         3d12h   10.1.0.53      docker-desktop   <none>           <none>
    etcd-docker-desktop                      1/1     Running   12         3d12h   192.168.65.4   docker-desktop   <none>           <none>
    kube-apiserver-docker-desktop            1/1     Running   12         3d12h   192.168.65.4   docker-desktop   <none>           <none>
    kube-controller-manager-docker-desktop   1/1     Running   12         3d12h   192.168.65.4   docker-desktop   <none>           <none>
    kube-proxy-7gdss                         1/1     Running   12         3d12h   192.168.65.4   docker-desktop   <none>           <none>
    kube-scheduler-docker-desktop            1/1     Running   17         3d12h   192.168.65.4   docker-desktop   <none>           <none>
    storage-provisioner                      1/1     Running   26         3d12h   10.1.0.51      docker-desktop   <none>           <none>
    vpnkit-controller                        1/1     Running   92         3d12h   10.1.0.50      docker-desktop   <none>           <none>

## Ejemplo minimo (uno.yml)

    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx
    spec:
        containers:
        - name: nginx
        image: nginx:alpine


## Aplicar manifiesto

    $ kubectl apply -f uno.yml
    pod/nginx created

    $ kubectl get pods -o wide
    NAME    READY   STATUS    RESTARTS   AGE    IP          NODE             NOMINATED NODE   READINESS GATES
    nginx   1/1     Running   0          3m1s   10.1.0.54   docker-desktop   <none>           <none>

## Ejecutar comando de un pod

    $ kubectl exec -it nginx -- sh

## Matar un pod

    $ kubectl delete pod nginx
    pod "nginx" deleted

    $ kubectl get pods -o wide
    No resources found in default namespace.

## Manifiesto más complejo ( dos.yml )

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    env:
    - name: MI_VARIABLE
      value: "pelado"
    - name: MI_OTRA_VARIABLE
      value: "pelade"
    - name: DD_AGENT_HOST
      valueFrom:
        fieldRef:
          fieldPath: status.hostIP
    resources:
      requests:
        memory: "64Mi"
        cpu: "200m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
    ports:
    - containerPort: 80
```

Ejecutar y obtener info


    $ kubectl apply -f uno.yml
        pod/nginx created


    $ kubectl get pods -o yaml
    apiVersion: v1
    items:
    - apiVersion: v1
    kind: Pod
    metadata:
        annotations:
        kubectl.kubernetes.io/last-applied-configuration: |
            {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"nginx","namespace":"default"},"spec":{"containers":[{"env":[{"name":"MI_VARIABLE","value":"pelado"},{"name":"MI_OTRA_VARIABLE","value":"pelade"},{"name":"DD_AGENT_HOST","valueFrom":{"fieldRef":{"fieldPath":"status.hostIP"}}}],"image":"nginx:alpine","livenessProbe":{"initialDelaySeconds":15,"periodSeconds":20,"tcpSocket":{"port":80}},"name":"nginx","ports":[{"containerPort":80}],"readinessProbe":{"httpGet":{"path":"/","port":80},"initialDelaySeconds":5,"periodSeconds":10},"resources":{"limits":{"cpu":"500m","memory":"128Mi"},"requests":{"cpu":"200m","memory":"64Mi"}}}]}}
        creationTimestamp: "2022-03-27T18:35:10Z"
        name: nginx
        namespace: default
        resourceVersion: "75654"
        uid: 1a839c98-6244-4474-9283-c075a04c1f67
    spec:
        containers:
        - env:
        - name: MI_VARIABLE
            value: pelado
        - name: MI_OTRA_VARIABLE
            value: pelade
        - name: DD_AGENT_HOST
            valueFrom:
            fieldRef:
                apiVersion: v1
                fieldPath: status.hostIP
        image: nginx:alpine
        imagePullPolicy: IfNotPresent
        livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 15
            periodSeconds: 20
            successThreshold: 1
            tcpSocket:
            port: 80
            timeoutSeconds: 1
        name: nginx
        ports:
        - containerPort: 80
            protocol: TCP
        readinessProbe:
            failureThreshold: 3
            httpGet:
            path: /
            port: 80
            scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
        resources:
            limits:
            cpu: 500m
            memory: 128Mi
            requests:
            cpu: 200m
            memory: 64Mi
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
            name: kube-api-access-nrtv4
            readOnly: true
        dnsPolicy: ClusterFirst
        enableServiceLinks: true
        nodeName: docker-desktop
        preemptionPolicy: PreemptLowerPriority
        priority: 0
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        serviceAccount: default
        serviceAccountName: default
        terminationGracePeriodSeconds: 30
        tolerations:
        - effect: NoExecute
        key: node.kubernetes.io/not-ready
        operator: Exists
        tolerationSeconds: 300
        - effect: NoExecute
        key: node.kubernetes.io/unreachable
        operator: Exists
        tolerationSeconds: 300
        volumes:
        - name: kube-api-access-nrtv4
        projected:
            defaultMode: 420
            sources:
            - serviceAccountToken:
                expirationSeconds: 3607
                path: token
            - configMap:
                items:
                - key: ca.crt
                path: ca.crt
                name: kube-root-ca.crt
            - downwardAPI:
                items:
                - fieldRef:
                    apiVersion: v1
                    fieldPath: metadata.namespace
                path: namespace
    status:
        conditions:
        - lastProbeTime: null
        lastTransitionTime: "2022-03-27T18:35:10Z"
        status: "True"
        type: Initialized
        - lastProbeTime: null
        lastTransitionTime: "2022-03-27T18:35:20Z"
        status: "True"
        type: Ready
        - lastProbeTime: null
        lastTransitionTime: "2022-03-27T18:35:20Z"
        status: "True"
        type: ContainersReady
        - lastProbeTime: null
        lastTransitionTime: "2022-03-27T18:35:10Z"
        status: "True"
        type: PodScheduled
        containerStatuses:
        - containerID: docker://3cbd526557a672de31460a882a45317392518a60c61405b5ac421221fac7a06e
        image: nginx:alpine
        imageID: docker-pullable://nginx@sha256:250c11e0c39dc17ba617f3eb0b28b6f456d46483f04823154c6aa68c432ded72
        lastState: {}
        name: nginx
        ready: true
        restartCount: 0
        started: true
        state:
            running:
            startedAt: "2022-03-27T18:35:11Z"
        hostIP: 192.168.65.4
        phase: Running
        podIP: 10.1.0.56
        podIPs:
        - ip: 10.1.0.56
        qosClass: Burstable
        startTime: "2022-03-27T18:35:10Z"
    kind: List
    metadata:
    resourceVersion: ""
    selfLink: ""