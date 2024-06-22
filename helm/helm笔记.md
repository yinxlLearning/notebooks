# helm



首先我们需要知道helm能够做什么？

### Helm 的主要功能

1.  **安装和管理 Kubernetes 应用**：
    -   Helm 可以简化 Kubernetes 应用程序的部署。通过 `helm install` 命令，你可以轻松地将复杂的应用程序安装到 Kubernetes 集群中。
    -   Helm Charts 包含了一个应用程序的所有 Kubernetes 资源定义（如 Deployment、Service、ConfigMap 等），允许你以一种结构化的方式管理这些资源。
2.  **版本管理**：
    -   Helm 支持应用程序版本管理，你可以使用 `helm upgrade` 来升级应用程序，也可以使用 `helm rollback` 回滚到之前的版本。
    -   这使得应用程序的升级和回滚操作非常简单和可靠。
3.  **参数化配置**：
    -   Helm Charts 支持参数化，你可以通过 `values.yaml` 文件或命令行参数传递自定义值，配置你的应用程序。
    -   这种参数化配置使得 Charts 变得非常灵活，可以在不同的环境中复用相同的 Charts。
4.  **打包和分享应用**：
    -   Helm 允许你打包并分享你的 Kubernetes 应用程序。你可以创建自己的 Charts 并发布到 Helm 仓库，供其他人使用。
    -   你也可以从公共 Helm 仓库（如 Artifact Hub）中下载和使用别人发布的 Charts。

### Helm 实现的功能

1.  **Helm Chart**：
    -   Helm Chart 是一个打包格式，包含了一个 Kubernetes 应用程序所需的所有资源定义和依赖项。
    -   一个 Chart 包括模板文件、默认值、Chart 描述文件等。
2.  **Helm Repository**：
    -   Helm Repository 是存放 Helm Charts 的地方。它类似于 Linux 的软件仓库。
    -   你可以添加、更新和搜索 Helm Repository 以找到所需的 Charts。
3.  **Release 管理**：
    -   Helm Release 是一个安装了的 Chart 的实例。每次你安装一个 Chart，Helm 会创建一个 Release。
    -   你可以管理这些 Releases，比如升级、回滚或删除它们。



## 基础



### 三大概念



-   Chart 代表 helm 包，包含在k8s集群内部运行应用程序，工具或服务所需的所有资源定义。
-   Repository 仓库，是用来存放盒共享charts的地方。就像maven的软件包仓库。
-   Release 是运行在 k8s 集群中 chart 的实例。一个 chart 通常可以在同一个集群中安装多次。每一次安装都会创建一个新的 *release*。

Helm 安装 *charts* 到 Kubernetes 集群中，每次安装都会创建一个新的 *release*。你可以在 Helm 的 chart *repositories* 中寻找新的 chart。



### helm search 查询charts包

Helm自带一个强大的搜索命令，可以用来从两种来源中进行搜索：

-   `helm search hub` 从 [Artifact Hub](https://artifacthub.io/) 中查找并列出 helm charts。 Artifact Hub中存放了大量不同的仓库。
-   `helm search repo` 从你添加（使用 `helm repo add`）到本地 helm 客户端中的仓库中进行查找。该命令基于本地数据进行搜索，无需连接互联网。



#### Helm添加仓库

`helm repo add` 命令的基本格式如下：

```bash
helm repo add <repo-name> <repo-url>
```

例如，添加 Prometheus 仓库：

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

#### 添加仓库的详细步骤和逻辑：

1.  **验证 URL**:
    -   Helm 会首先验证提供的 URL 是否有效，并检查它是否指向一个合法的 Helm 仓库。一个合法的 Helm 仓库需要包含一个 `index.yaml` 文件，描述该仓库中所有可用的 Charts。
2.  **获取仓库索引**:
    -   Helm 从指定的 URL 下载 `index.yaml` 文件。这是一个 YAML 文件，包含了仓库中所有可用 Charts 的元数据，包括每个 Chart 的名称、版本、描述等信息。
3.  **更新本地配置**:
    -   Helm 会将仓库的 URL 和名称保存到本地的配置文件中，通常是 `$HELM_HOME/repository/repositories.yaml` 文件。这个文件包含了所有已添加仓库的列表及其 URL。
4.  **存储仓库索引**:
    -   下载的 `index.yaml` 文件被保存在本地缓存中，以便在用户搜索 Charts 或查看仓库内容时能够快速响应。这个文件通常被保存在 `$HELM_HOME/repository/cache/` 目录下，文件名是根据仓库名称生成的，如 `repository/cache/prometheus-community-index.yaml`。
5.  **刷新缓存（可选）**:
    -   在添加仓库后，可以运行 `helm repo update` 命令来刷新所有已添加仓库的本地缓存，确保缓存中的 `index.yaml` 文件是最新的。



## 创建你自己的 charts

[chart 开发指南](https://helm.sh/zh/docs/topics/charts) 介绍了如何开发你自己的chart。 但是你也可以通过使用 `helm create` 命令来快速开始：

```console
$ helm create deis-workflow
Creating deis-workflow
```

现在，`./deis-workflow` 目录下已经有一个 chart 了。你可以编辑它并创建你自己的模版。

在编辑 chart 时，可以通过 `helm lint` 验证格式是否正确。

当准备将 chart 打包分发时，你可以运行 `helm package` 命令：

```console
$ helm package deis-workflow
deis-workflow-0.1.0.tgz
```

然后这个 chart 就可以很轻松的通过 `helm install` 命令安装：

```console
$ helm install deis-workflow ./deis-workflow-0.1.0.tgz
...
```

打包好的 chart 可以上传到 chart 仓库中。查看 [Helm chart 仓库](https://helm.sh/zh/docs/topics/chart_repository)获取更多信息。





### helm create xx 创建的目录



```shell
mychart/
├── charts/
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests/
│       └── test-connection.yaml
├── Chart.yaml 
└── values.yaml
```



### 目录和文件详解

1.  **Chart.yaml**

    -   这是 Helm Chart 的元数据文件，包含关于 Chart 的基本信息，如名称、版本、描述等。

    -   示例内容：

        ```yaml
        # 指定 Chart 的 API 版本。在 Helm 3 中，常用的版本为 v2。它决定了 Chart 的结构和内容的约束规则。
        # 确保 Helm 工具能够正确解析和处理 Chart 文件。
        apiVersion: v2
        # chart 的名称，唯一标识此chart，通常与应用程序的名称一致
        name: mychart
        # 对chart的简单描述
        description: A Helm chart for Kubernetes
        # Chart的类型，可以是application或library
        # application 表示这是一个应用程序 chart，包含可以部署的模板
        # library 表示这是一个库 chart，提供有用的函数或工具，不可以单独部署
        type: application
        # chart 的版本号，遵循语义化版本控制，在每次对 Chart 或其模板进行更改时应递增此版本号。用于版本控制和管理。
        version: 0.1.0
        # 应用程序的版本号，反映部署的应用程序的版本。此版本号不需要遵循语义化版本控制规则，可以使用任何格式，建议用引号括起来。用于告知用户应用程序的实际版本。
        appVersion: 1.16.0
        ```

2.  **values.yaml**

    -   默认的配置文件，包含所有模板中可以使用的变量的默认值。用户可以在安装或升级 Chart 时通过 `-f` 参数覆盖这些值。

    -   示例内容：

        ```yaml
        # Default values for deis-workflow.
        # This is a YAML-formatted file.
        # Declare variables to be passed into your templates.
        
        # 指定应用程序副本的数量。用于在deployment中设置副本数量
        replicaCount: 1
        
        # 定义容器镜像相关的参数
        image:
          # 仓库地址
          # 仓库地址的定义策略取决于你希望从哪个容器注册表（容器仓库）拉取镜像
          # 如果使用公共容器注册表（如 Docker Hub），通常可以直接写镜像名称或带有命名空间的镜像名称。 
          #   如 repository: nginx，这将从 Docker Hub 拉取 nginx 镜像，使用 stable 标签。
          #   如果镜像在一个特定的命名空间中，例如 library/nginx（Docker Hub 的官方镜像），你也可以这样写 repository: library/nginx
          # 如果使用私有容器注册表，通常需要指定完整的仓库地址，包括注册表的域名、命名空间和镜像名称。
          #   如：repository: myregistry.example.com/mynamespace/myapp
          # 云提供商通常有自己的容器注册表服务，如 AWS ECR、GCP GCR、Azure ACR 等。需要提供完整的路径。如repository: 123456789012.dkr.ecr.us-west-1.amazonaws.com/myapp
          # 如果使用本地私有注册表，也需要指定完整的路径。 repository: localhost:5000/myapp tag: v1.0.0
          #   解释：这将从运行在 localhost 上的私有注册表中拉取 myapp 镜像，使用 v1.0.0 标签。
          # 在某些情况下，你可能希望将 Helm Chart 的名称作为镜像名称的一部分，这通常适用于公共容器注册表和私有命名空间。
          #   如：repository: mychart 假设 mychart 是一个公共镜像（例如在 Docker Hub 上），这将直接拉取名为 mychart 的镜像。
          repository: nginx
          # 镜像录取策略 Always,IfNotPresent,Never
          pullPolicy: IfNotPresent
          # Overrides the image tag whose default is the chart appVersion.
          # 镜像标签
          tag: ""
        
        # 镜像拉取密钥，用于访问私有仓库
        # 用来配置k8s中的secret名称，比如
        # imagePullSecrets:
        #   - name: my-registry-secret
        imagePullSecrets: []
        
        # 覆盖默认的 Chart 名称
        nameOverride: ""
        
        # 覆盖默认的 Chart 全名
        # 用于覆盖Helm Charts中定义的默认名称
        # 这将覆盖 Helm 默认的完整名称，例如将完整名称从 {{ .Release.Name }}-{{ .Chart.Name }} 改为 my-fullname。
        fullnameOverride: "my-fullname"
        
        # ServiceAccount 相关配置
        serviceAccount:
          # 是否创建 ServiceAccount
          create: true
          # 自动挂载 ServiceAccount 的 API 凭证
          automount: true
          # 给 ServiceAccount 添加注解
          annotations: {}
          # 使用的 ServiceAccount 名称。如果未设置且 create 为 true，会使用 fullname 模板生成一个名称
          name: ""
        
        # 给 Pod 添加注解
        podAnnotations: {}
        # 给 Pod 添加标签
        podLabels: {}
        
        # Pod 安全上下文配置
        podSecurityContext: {}
          # fsGroup: 2000
        
        # 容器安全上下文配置
        securityContext: {}
          # capabilities:
          #   drop:
          #   - ALL
          # readOnlyRootFilesystem: true
          # runAsNonRoot: true
          # runAsUser: 1000
        
        # Service 相关配置
        service:
          # Service 类型（如 ClusterIP, NodePort, LoadBalancer）
          type: ClusterIP
          # Service 暴露的端口
          port: 80
        
        # Ingress 相关配置
        ingress:
          # 是否启用 Ingress
          enabled: false
          className: ""
          # Ingress 注解
          annotations: {}
            # kubernetes.io/ingress.class: nginx
            # kubernetes.io/tls-acme: "true"
          # Ingress 主机配置
          hosts:
            - host: chart-example.local
              paths:
                - path: /
                  pathType: ImplementationSpecific
          # Ingress TLS 配置
          tls: []
          #  - secretName: chart-example-tls
          #    hosts:
          #      - chart-example.local
        
        # 资源请求和限制配置
        resources: {}
          # limits:
          #   cpu: 100m
          #   memory: 128Mi
          # requests:
          #   cpu: 100m
          #   memory: 128Mi
        
        # livenessProbe 配置，用于检测 Pod 是否处于健康状态
        livenessProbe:
          httpGet:
            path: /
            port: http
        
        # readinessProbe 配置，用于检测 Pod 是否已准备好接受流量
        readinessProbe:
          httpGet:
            path: /
            port: http
        
        # 自动扩缩容相关配置
        autoscaling:
          # 是否启用自动扩缩容
          enabled: false
          # 最小副本数
          minReplicas: 1
          # 最大副本数
          maxReplicas: 100
          # CPU 利用率目标
          targetCPUUtilizationPercentage: 80
          # targetMemoryUtilizationPercentage: 80
        
        # 额外的卷定义
        volumes: []
        # - name: foo
        #   secret:
        #     secretName: mysecret
        #     optional: false
        
        # 额外的卷挂载定义
        volumeMounts: []
        # - name: foo
        #   mountPath: "/etc/foo"
        #   readOnly: true
        
        # 节点选择器，用于指定 Pod 调度到哪些节点
        nodeSelector: {}
        
        # 容忍度配置，用于指定 Pod 可以容忍哪些污点
        tolerations: []
        
        # 亲和性配置，用于指定 Pod 的调度策略
        affinity: {}
        
        ```

        -   

3.  **charts/** 目录

    -   这个目录用来存放 Chart 的依赖项。如果你的 Chart 依赖其他 Charts，可以将它们的 `.tgz` 压缩包放在这个目录中。

4.  **templates/** 目录

    -   存放 Kubernetes 资源的模板文件，这些模板会被渲染成实际的 Kubernetes 资源定义。
    -   常见模板文件：
        -   `deployment.yaml`：定义 Kubernetes Deployment 资源。
        -   `service.yaml`：定义 Kubernetes Service 资源。
        -   `ingress.yaml`：定义 Kubernetes Ingress 资源。
        -   `hpa.yaml`：定义 Kubernetes HorizontalPodAutoscaler 资源。
        -   `serviceaccount.yaml`：定义 Kubernetes ServiceAccount 资源。
        -   `_helpers.tpl`：存放模板辅助函数，通常用于定义模板内的变量或通用的片段。
        -   `tests/test-connection.yaml`：定义测试资源，用于安装后验证应用是否正常工作。

### templates 目录中的文件详解

1.  **_helpers.tpl**

    -   存放模板辅助函数和局部模板，通常用于定义模板变量或通用的模板片段，便于在其他模板文件中复用。

    -   示例内容：

        ```
        tpl
        复制代码
        {{- define "mychart.labels" -}}
        app.kubernetes.io/name: {{ include "mychart.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
        app.kubernetes.io/version: {{ .Chart.AppVersion }}
        app.kubernetes.io/managed-by: {{ .Release.Service }}
        {{- end -}}
        ```

2.  **deployment.yaml**

    -   定义 Kubernetes Deployment 资源，用于管理应用程序的副本和部署策略。

    -   示例内容：

        ```
        yaml
        复制代码
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: {{ include "mychart.fullname" . }}
          labels:
            {{ include "mychart.labels" . | nindent 4 }}
        spec:
          replicas: {{ .Values.replicaCount }}
          selector:
            matchLabels:
              {{ include "mychart.selectorLabels" . | nindent 6 }}
          template:
            metadata:
              labels:
                {{ include "mychart.selectorLabels" . | nindent 8 }}
            spec:
              containers:
                - name: {{ .Chart.Name }}
                  image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
                  imagePullPolicy: {{ .Values.image.pullPolicy }}
                  ports:
                    - containerPort: 80
                  livenessProbe:
                    httpGet:
                      path: /
                      port: 80
                  readinessProbe:
                    httpGet:
                      path: /
                      port: 80
        ```

3.  **service.yaml**

    -   定义 Kubernetes Service 资源，用于暴露应用程序的网络服务。

    -   示例内容：

        ```
        yaml
        复制代码
        apiVersion: v1
        kind: Service
        metadata:
          name: {{ include "mychart.fullname" . }}
          labels:
            {{ include "mychart.labels" . | nindent 4 }}
        spec:
          type: {{ .Values.service.type }}
          ports:
            - port: {{ .Values.service.port }}
              targetPort: 80
              protocol: TCP
              name: http
          selector:
            {{ include "mychart.selectorLabels" . | nindent 4 }}
        ```

4.  **ingress.yaml**

    -   定义 Kubernetes Ingress 资源，用于配置外部访问到应用程序的 HTTP 和 HTTPS 路由。

    -   示例内容：

        ```
        yaml
        复制代码
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: {{ include "mychart.fullname" . }}
          labels:
            {{ include "mychart.labels" . | nindent 4 }}
          annotations:
            {{- range $key, $value := .Values.ingress.annotations }}
              {{ $key }}: {{ $value | quote }}
            {{- end }}
        spec:
          rules:
            - host: {{ .Values.ingress.hosts[0].host }}
              http:
                paths:
                  - path: /
                    pathType: ImplementationSpecific
                    backend:
                      service:
                        name: {{ include "mychart.fullname" . }}
                        port:
                          number: 80
        ```

5.  **serviceaccount.yaml**

    -   定义 Kubernetes ServiceAccount 资源，用于提供应用程序的身份验证。

    -   示例内容：

        ```
        yaml
        复制代码
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: {{ include "mychart.fullname" . }}
          labels:
            {{ include "mychart.labels" . | nindent 4 }}
        ```

6.  **tests/test-connection.yaml**

    -   定义测试资源，用于安装后验证应用是否正常工作。通常是一个简单的测试 Pod。

    -   示例内容：

        ```
        yaml
        复制代码
        apiVersion: v1
        kind: Pod
        metadata:
          name: "{{ include "mychart.fullname" . }}-test-connection"
          labels:
            {{ include "mychart.labels" . | nindent 4 }}
        spec:
          containers:
            - name: wget
              image: busybox
              command: ['wget']
              args: ['{{ include "mychart.fullname" . }}:{{ .Values.service.port }}']
        ```

### 其他注意事项

-   **Linting**：使用 `helm lint` 检查你的 Chart 语法是否正确。
-   **Packaging**：使用 `helm package <chart-directory>` 打包你的 Chart。
-   **Publishing**：将打包好的 Chart 上传到 Helm 仓库。

通过 `helm create` 创建的目录结构和文件，提供了一个基础框架，帮助你组织和管理 Kubernetes 应用程序的配置文件，并确保它们符合最佳实践。你可以根据实际需求修改和扩展这些文件，以便更好地适应你的应用程序和环境。

