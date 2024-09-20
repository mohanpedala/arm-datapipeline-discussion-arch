## Kafka Ingress Sample Config
* Traffic will flow through
    LB -> Istiogateway -> nginx -> kafka bootstrap
                                -> kafka broker-N

* Expose the brokers
    ```yaml
        listeners:
      - name: external
        port: 9094
        type: ingress
        tls: false
        configuration:
          bootstrap:
            host: my-kafka-cluster-bootstrap.example.com
          brokers:
            - broker: 0
              advertisedHost: my-kafka-cluster-broker-0.example.com
            - broker: 1
              advertisedHost: my-kafka-cluster-broker-1.example.com
            - broker: 2
              advertisedHost: my-kafka-cluster-broker-2.example.com
    ```

* Virtual service
    ```yaml
    apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
    name: kafka-virtual-service
    namespace: istio-system
    spec:
    hosts:
    - "kafka.example.com"
    gateways:
    - kafka-gateway
    http:
    - match:
        - uri:
            prefix: /kafka
        route:
        - destination:
            host: nginx-ingress.default.svc.cluster.local
            port:
            number: 80
    ```

 * Ingress
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: kafka-nginx-ingress
    namespace: kafka-cluster
    annotations:
        nginx.ingress.kubernetes.io/ssl-redirect: "false"  
    spec:
        ingressClassName: nginx # Tells Kubernetes to route via the NGINX controller
        rules:
            - host: my-kafka-cluster-bootstrap-nginx.example.com
            http:
                paths:
                - path: /
                    pathType: Prefix
                    backend:
                    service:
                        name: my-kafka-cluster-kafka-bootstrap
                        port:
                        number: 9094
    ```