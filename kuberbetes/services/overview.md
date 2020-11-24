### Service
- In Kubernetes, a Service is an abstraction which defines a logical set of Pods and a policy by which to access them

ServiceType:
    default --> ClusterIP

    ClusterIP: 在一个clusterip上暴露服务，这个类型只能在集群内部被识别
    NodePort: 在k8s节点的所有IP地址上暴露服务，0.0.0.0:port,    
        port --> 与service-node-port-range相关
        ip --> --nodeport-addresses 相关 
            --nodeport-addresses=127.0.0.0/8 将会自动选择 127.0.0.1
            apiVersion: v1
            kind: Service
            metadata:
            name: external
            spec:
            type: NodePort
            selector:
                app: myapp
            ports:
            - port: 80
                targetPort: 80
                nodePort: 31080
    LoadBalancer: Exposes the Service externally using a cloud provider's load balancer.
    ExternaName:Maps the Service to the contents of the externalName field 
        (e.g. foo.bar.example.com), by returning a CNAME record