# ConfigMap

#### Create a ConfigMap

You can use either kubectl create configmap or ConfigMap generator in kustomizaton.yaml to create a configmap

```
kubectl create configmap <map-name> <data-source>
<data-source> --> file, directory, literal value draw the data from.
当我们以文件为data-source时，<data-source>中的键默认为文件的基本名称，而值默认为文件内容。
```

Create a configmap from directories

```
kubectl create configmap game-config --from-file=/path/to/dir

mkdir -p configure-pod-container/configmap/

# Download the sample files into `configure-pod-container/configmap/` directory
wget https://kubernetes.io/examples/configmap/game.properties -O configure-pod-container/configmap/game.properties
wget https://kubernetes.io/examples/configmap/ui.properties -O configure-pod-container/configmap/ui.properties

kubectl create configmap game-config --from-file=configure-pod-container/configmap/


```
The above command packages each files,
可以查看生成的configmap-->
```
kubectl describe configmaps game-config

Name:         game-config // configmap的名称
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data //configmap的内容
====
game.properties: // 文件名
---- // 从这里开始是文件的内容
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
ui.properties: // 文件名
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice
```

kubectl 命令行创建的configmap 等于下面 -->
``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  creationTimestamp: 2016-02-18T18:52:05Z
  name: game-config
  namespace: default
  resourceVersion: "516"
  uid: b4952dc3-d670-11e5-8cd0-68f728db1985
data:
  game.properties: |
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
  ui.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    how.nice.to.look=fairlyNice
```


#### Create ConfigMaps from files

```
kubectl create configmap file-config --from-file=configure-pod-container/configmap/game.properties
```

从文件中创建，只会含有一个文件和内容
不过可以执行很多次，并且--from-file如果指向别的文件的话， 送有的内容都会追加进去


#### Use the option --from-env-file to create a ConfigMap from an env-file, for example:
```
Env-file 包含了很多的键值对，开头为"#"的行，空白的行将会被忽略
examples:

cat configure-pod-container/configmap/game-env-file.properties
enemies=aliens
lives=3
allowed="true"

kubectl create configmap game-config-env-file \
       --from-env-file=configure-pod-container/configmap/game-env-file.properties
```

我们查看configmap 的详细信息
```yaml
data:
  color: purple
  how: fairlyNice
  textmode: "true"
```

#### Define the key to use when creating a ConfigMap from a file 

我们可以在创建configmap的时候修改data字段列表的名称
```
kubectl create configmap config-map-1 --from-file=<key-name>=</path/to/file>
```
example: -->
```
kubectl create configmap my-config-map --from-file=my-key=/config/xxx.yaml

kubectl get configmaps my-config-map -o yaml -->
...
data:
  my-key: |
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
```

#### Create configmap from literal values.

We can use --from-literal define the literal values from command line:
```
kubectl create configmap my-config --from-literal=<key>=<value>

kubectl create configmap my-config --from-literal=special.value=camellia --from-literal=special.color=pink

kubectl get ... -o yaml
...
data:
  special.value: camellia
  special.color: pink
```

#### Create a ConfigMap from Generator

    kubectl support kustomization.yaml since 1.14. We can alse create a configmap from generators and then apply
it to create object on the Apiserver.
    kustomization.yaml should be specified in a directory with many files.
example: -->
```text
cat server/config.json
{
    "name":"camellia"
}

cat game/blue
color=blue

cat game/pink
color=pink

cat game/white
color=white
--------
利用kubectl apply -k . 创建configmap:
# cat kustomization.yaml
configMapGenerator:
- name: wocao-haha
  files:
  - server/config.json
- name: dingni-xixi
  files:
  - [Custom-defined-key --> <keyName>=]game/blue
  # 我们也可以在其中添加自定义的key
  - game/pink
  - game/white
- name: lamp-heihei # 也可以直接定义literal类型的内容
  literals:
  - special.color=blue
  - special.flower=orchid
  - special.generation=four

Then apply:
kubectl apply -k .

output --> /path/to/config-file created
```

我们看看其中的细节
使用kustomization.yaml 生成的configmap会带一个随机的hash值
kubectl get configmap
NAME                              DATA   AGE
dingni-xixi-277kdf4g9m            3      11s
wocao-haha-hcg2452885             1      11s

如果kustomization.yaml的一个configmap中含有多个files字段时, 会变成如下
```
data:
  blue: |
    color=blue
  pink: |
    color=pink
  white: |
    color=white
kind: ConfigMap
```

#Use ConfigMap

1. Define container environment variables using configMap data
```
    kubectl create configmap env-map --from-literal=special.color=blue
```
  将special.color的值定义在pod的yaml文件中, 并且分配给指定的环境变量
```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    name: new-env-pod
    spec:
    containers:
    - name: new-env-container
      image: busybox:latest
      command: ["/bin/sh","-c","env"]
      env:
        # name 定义了环境变量在container中的名称
        - name: SPECIAL_COLOR
          valueFrom:
            configMapKeyRef:
              name: env-map
              key: special.color
    restartPolicy: Never
```

2. Define container environment variables with data from multiple ConfigMaps 
``` yaml
    spec: 
    containers:
    - name: newcontainer
        image: busybox:latest
        command: ["/bin/sh","-c","env"]
        env:
        - name: SPECIAL_COLOR
            valueFrom:
                configMapKeyRef:
                name: CONFIG_MAP_NAME
                key: configmap.data.字段
        - name: SPECIAL_LOG_LEVEL
            valueFrom:
                configMapKeyRef:
                name: CONFIG_MAP_NAME
                key: configmap.data.字段
        ...
```

3. config all key-value pairs in a configmap as container env variables

```
envFrom:
- configMapRef:
    name: CONFIGMAP_NAME
```

# ADD ConfigMap data to volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-vlume
      configMap:
        name: haha
  restartPolicy: Never

比如是configmap haha --from-literal=special.color=blue --> 
会在/etc/config/目录底下生成文件名为special.color, 内容为blue的文件
configmap 的key会成为文件的名称

指定挂载后的名称 -->
volumes:
  - name: config-volume
    configMap:
      name: haha
      items:
      - key: special.color //configmap的key
        path: keys // 传递到容器中的名称
ls /etc/config 
keys

# 当我们使用--from-file 创建的configmap时，如果没有指定key, 则默认的key就是文件名称
  可以使用 --from-file=<keyname>=filename 指定key
```

#### 热更configmap
```
可以使用 kubectl edit configmap CONFIGMAP_NAME
修改后的configmap 无需重新挂载，k8s会自动检测configmap的变化，并修改
```
