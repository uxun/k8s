## Custom Charts

[refernece](https://github.com/helm/helm/blob/master/docs/charts.md)

[Go Template](https://golang.org/pkg/text/template/)

[sprig](http://masterminds.github.io/sprig/)

> - NOTES.txt：helm install 文本信息
> - deployment.yaml：kubernetes 应用配置清单
> - service.yaml：kubernetes service 配置清单
> - ingress.yaml: kubernetes ingress 对象的资源清单文件
> - _helpers.tpl：当前 chart 中重复使用
>
> 需要注意的是`kubernetes`资源对象的 labels 和 name 定义被[限制 63个字符](http://kubernetes.io/docs/user-guide/labels/#syntax-and-character-set)，所以需要注意名称的定义。

```shell
$ helm create testchart
$ tree testchart/
testchart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

```

Run debug

```shell
# modify .yaml
$ helm install . --dry-run --debug ./testchart

# --set 优先级最高
$ helm install --dry-run --debug --set course=python ./testchart
```

#### 内置对象 [REFERNECE](<https://github.com/helm/helm/blob/6ceaef446f70ac125b9a6d02b1a7a46956b104af/docs/charts.md#predefined-values>)

> Helm 常用内置对象，GO 模板命名约定 内置值始终首字母大写

##### 1.Chart [refernece](https://github.com/helm/helm/blob/master/docs/charts.md#the-chartyaml-file)

```
Chart.yaml文件中内容，Chart对象从Chart.yaml文件中读取 Refernece
```

##### 2.Release

```
Release.Name：release 名称
Release.Time：release 的时间
Release.Namespace：release 的 namespace（如果清单未覆盖）
Release.Service：release 服务的名称（始终是 Tiller）。
Release.Revision：此 release 的修订版本号，从1开始累加。
Release.IsUpgrade：如果当前操作是升级或回滚，则将其设置为 true。
Release.IsInstall：如果当前操作是安装，则设置为 true。
```

##### 3.Values

```
values.yaml文件和用户提供的文件传入模板的值，默认null
```

##### 4.Global Values

```
# values.yaml 定义全局值 通过{{ .Values.global.app }} 引用
global:
  app: MyWordPress
```

#### FUNC(函数) [REFERNECE](https://github.com/helm/helm/blob/master/docs/charts_tips_and_tricks.md)

> {{ .Values.course.k8s | upper | quote }}
>
> e.g k8s = uxun
>
> e.g quote：转为："uxun"
>
> e.g upper：转为：UXUN
>
> e.g default： {{ .Values.you | default  "My" | quote }} you无值，设置为"My"

#### Flow Control (控制流) [REFERNECE](https://github.com/helm/helm/blob/master/docs/chart_template_guide/control_structures.md)

if

> {{- if eq .Values.course.python "django" }}
> web: true
> {{- end }}
>
> -：空格控制 

with：

> 变量作用域  {{- with .Values.course }} xxx {{- end}} 
>
> 使用with将作用域指向.Values.course : value.yaml

range

> 该函数将会遍历{{ .Values.courselist }}列表
>
> courselist:
>
> {{- range .Values.courselist }}
>
> \- {{ . | title | quote }}
> {{- end }}

#### Named Templates (命名模板) [REFERNECE](https://github.com/helm/helm/blob/7ad9d7b766af839a1d271a5394f194cfd2478b43/docs/chart_template_guide/named_templates.md)

> define：关键字声明命名模板，全局可访问
>
> e.g:
> {{ define "MY.NAME" }}
> \# body of template here
> {{ end }}

```yaml
# create _helpers.tpl
{{/* Generate basic labels */}}
{{- define "mychart.labels" }}
  labels:
    generator: helm
    date: {{ now | htmlDate }}
{{- end }}

# mychart/templates/configmap.yaml
# "."配置顶层作用域  {{- template "mychart.labels" . }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  {{- template "mychart.labels" }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
  
# $ helm install --dry-run --debug .
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: running-panda-configmap
  labels:
    generator: helm
    date: 2019-04-29
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
```

##### The `include` function

> 模板不是函数而是一个动作，无法将模板调用的输出传递给其他函数，数据只是插入内联。
>
> 通过include 函数通过管道控制格式

e.g

```yaml
# 定义_helpers.tpl 
{{- define "mychart.app" -}}
app_name: {{ .Chart.Name }}
app_version: "{{ .Chart.Version }}+{{ .Release.Time.Seconds }}"
{{- end -}}
# 使用 include 
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
  labels:
    {{- include "mychart.app" . | nindent 4 }}
data:
  myvalue: "Hello World"
  {{- range $key, $val := .Values.favorite }}
  {{ $key }}: {{ $val | quote }}
  {{- end }}
  {{- include "mychart.app" . | nindent 2 }}
```

#### Hooks [REFERNECE](https://github.com/helm/helm/blob/b06b5ef4d2396e0809af530da6bab180517e4ea6/docs/charts_hooks.md#hooks)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{.Release.Name}}"
  labels:
    app.kubernetes.io/managed-by: {{.Release.Service | quote }}
    app.kubernetes.io/instance: {{.Release.Name | quote }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    helm.sh/chart: "{{.Chart.Name}}-{{.Chart.Version}}"
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      name: "{{.Release.Name}}"
      labels:
        app.kubernetes.io/managed-by: {{.Release.Service | quote }}
        app.kubernetes.io/instance: {{.Release.Name | quote }}
        helm.sh/chart: "{{.Chart.Name}}-{{.Chart.Version}}"
    spec:
      restartPolicy: Never
      containers:
      - name: post-install-job
        image: "alpine:3.3"
        command: ["/bin/sleep","{{default "10" .Values.sleepyTime}}"]
```



```
annotation:
  prometheus.io/scrape: 'true' # 自动发现服务
```

