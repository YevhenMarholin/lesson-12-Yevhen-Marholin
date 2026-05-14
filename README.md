# lesson-12-Yevhen-Marholin

---

## 1. Встановлення Dragonfly Operator

Було встановлено Dragonfly Operator:

```bash
 kubectl apply -f https://raw.githubusercontent.com/dragonflydb/dragonfly-operator/main/manifests/dragonfly-operator.yaml
namespace/dragonfly-operator-system created
customresourcedefinition.apiextensions.k8s.io/dragonflies.dragonflydb.io created
serviceaccount/dragonfly-operator-controller-manager created
role.rbac.authorization.k8s.io/dragonfly-operator-leader-election-role created
clusterrole.rbac.authorization.k8s.io/dragonfly-operator-manager-role created
clusterrole.rbac.authorization.k8s.io/dragonfly-operator-metrics-reader created
clusterrole.rbac.authorization.k8s.io/dragonfly-operator-proxy-role created
rolebinding.rbac.authorization.k8s.io/dragonfly-operator-leader-election-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/dragonfly-operator-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/dragonfly-operator-proxy-rolebinding created
service/dragonfly-operator-controller-manager-metrics-service created
deployment.apps/dragonfly-operator-controller-manager created
```

Перевірка:

```bash
kubectl get pods -n dragonfly-operator-system
```

Очікуваний результат:

```bash
NAME                                                    READY   STATUS    RESTARTS   AGE
dragonfly-operator-controller-manager-b9ff77cdf-s98tf   2/2     Running   0          25s
```

---

## 2. Перевірка CRD

Було перевірено появу Custom Resource Definitions:

```bash
kubectl api-resources | grep dragonfly
```

Результат:

```bash
dragonflies                                      dragonflydb.io/v1alpha1           true         Dragonfly```

---

## 3. Аналіз Dragonfly spec

Для перегляду доступних полів було виконано:

```bash
kubectl explain dragonfly.spec

GROUP:      dragonflydb.io
KIND:       Dragonfly
VERSION:    v1alpha1

FIELD: spec <Object>


DESCRIPTION:
    DragonflySpec defines the desired state of Dragonfly
    
FIELDS:
  aclFromSecret <Object>
    (Optional) Acl file Secret to pass to the container

  additionalContainers  <[]Object>
    (Optional) Additional containers to add to dragonflycluster. Replace
    container on name collision.

  additionalVolumes     <[]Object>
    (Optional) Additional volumes to add to dragonflycluster. Replace volume on
    name collision.

  affinity      <Object>
    (Optional) Dragonfly pod affinity

  annotations   <map[string]string>
    (Optional) Annotations to add to the Dragonfly pods.

  args  <[]string>
    (Optional) Dragonfly container args to pass to the container
    Refer to the Dragonfly documentation for the list of supported args

  authentication        <Object>
    (Optional) Dragonfly Authentication mechanism

  containerSecurityContext      <Object>
    (Optional) Dragonfly container security context

  enableReplicationReadinessGate        <boolean>
    (Optional) When enabled, adds a custom readiness gate to pods that prevents
    them from being considered Ready until replication is fully synced.
    Protects against data loss during external node drains.
    WARNING: Enabling this on an existing cluster will trigger a rolling update.

  env   <[]Object>
    (Optional) Env variables to add to the Dragonfly pods.

  image <string>
    Image is the Dragonfly image to use

  imagePullPolicy       <string>
    (Optional) imagePullPolicy to set to Dragonfly, default is Always

  imagePullSecrets      <[]Object>
    (Optional) imagePullSecrets to set to Dragonfly

  initContainers        <[]Object>
    (Optional) Dragonfly pod init containers

  labels        <map[string]string>
    (Optional) Labels to add to the Dragonfly pods.

  memcachedPort <integer>
    (Optional) Dragonfly memcached port

  networkPolicyEnabled  <boolean>
    (Optional) Whether to create a NetworkPolicy for this Dragonfly instance.
    The NetworkPolicy restricts admin port access to operator and peer pods
    only.
    Defaults to true.

  nodeSelector  <map[string]string>
    (Optional) Dragonfly pod node selector

  ownedObjectsMetadata  <Object>
    (Optional) Dragonfly direct child resources additional annotations and
    labels

  pdb   <Object>
    (Optional) Dragonfly Pod Disruption Budget configuration

  podSecurityContext    <Object>
    (Optional) Dragonfly pod security context

  priorityClassName     <string>
    (Optional) Dragonfly pod priority class name

  replicas      <integer>
    Replicas is the total number of Dragonfly instances including the master

  resources     <Object>
    (Optional) Dragonfly container resource limits. Any container limits
    can be specified.

  serviceAccountName    <string>
    (Optional) Dragonfly pod service account name

  serviceSpec   <Object>
    (Optional) Dragonfly Service configuration

  skipFSGroup   <boolean>
    (Optional) Skip Assigning FileSystem Group. Required for platforms such as
    Openshift that require IDs to not be set, as it injects a fixed randomized
    ID per namespace into all pods.

  snapshot      <Object>
    (Optional) Dragonfly Snapshot configuration

  tiering       <Object>
    (Optional) Dragonfly SSD Tiering configuration

  tlsSecretRef  <Object>
    (Optional) Dragonfly TLS secret to used for TLS
    Connections to Dragonfly. Dragonfly instance  must
    have access to this secret and be in the same namespace

  tolerations   <[]Object>
    (Optional) Dragonfly pod tolerations

  topologySpreadConstraints     <[]Object>
    (Optional) Dragonfly pod topologySpreadConstraints
```

---

## 4. Створення Dragonfly instance

Було створено файл:

```text
dragonfly.yaml
```

```yaml
apiVersion: dragonflydb.io/v1alpha1
kind: Dragonfly
metadata:
  name: dragonfly
spec:
  replicas: 1
  image: docker.dragonflydb.io/dragonflydb/dragonfly:latest
```

Застосування:

```bash
kubectl apply -f dragonfly.yaml

dragonfly.dragonflydb.io/dragonfly created
```

Перевірка:

```bash
kubectl get dragonflies
kubectl get pods
kubectl get svc

NAME        PHASE              ROLLING UPDATE   REPLICAS
dragonfly   ResourcesCreated                    1
NAME                                     READY   STATUS    RESTARTS        AGE
course-app-course-app-6f44487c4f-9cbk4   1/1     Running   3 (6m45s ago)   3d21h
course-app-course-app-6f44487c4f-ff2cd   1/1     Running   3 (6m46s ago)   3d21h
course-app-course-app-6f44487c4f-l5sln   1/1     Running   3 (6m41s ago)   3d21h
dragonfly-0                              1/1     Running   1               14s
redis-master-0                           1/1     Running   1 (7m48s ago)   3d21h
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
course-app-course-app   ClusterIP   10.96.233.233   <none>        8080/TCP   3d21h
dragonfly               ClusterIP   10.96.237.75    <none>        6379/TCP   14s
kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP    3d22h
redis-headless          ClusterIP   None            <none>        6379/TCP   3d21h
redis-master            ClusterIP   10.96.214.28    <none>        6379/TCP   3d21h
```

```

---

## 5. Перевірка Dragonfly

Було перевірено роботу Dragonfly через `redis-cli`:

```bash
kubectl run -it --rm redis-cli \
  --image=redis:7-alpine \
  --restart=Never \
  -- redis-cli -h dragonfly.default ping
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
warning: couldn't attach to pod/redis-cli, falling back to streaming logs: Internal error occurred: unable to upgrade connection: container redis-cli not found in pod redis-cli_default
PONG
pod "redis-cli" deleted from default namespaceAll commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
warning: couldn't attach to pod/redis-cli, falling back to streaming logs: Internal error occurred: unable to upgrade connection: container redis-cli not found in pod redis-cli_default
PONG
pod "redis-cli" deleted from default namespace

```

Очікувана відповідь:

```bash
PONG
```

---

## 6. Інтеграція course-app з Dragonfly

У `values.yaml` Helm chart було оновлено змінні середовища:

```yaml
env:
  APP_STORE: redis
  APP_REDIS_URL: redis://dragonfly:6379
```

Оновлення Helm release:

```bash
helm upgrade course-app ./course-app-chart
Release "course-app" has been upgraded. Happy Helming!
NAME: course-app
LAST DEPLOYED: Thu May 14 12:39:33 2026
NAMESPACE: default
STATUS: deployed
REVISION: 3
DESCRIPTION: Upgrade complete
TEST SUITE: None
```

Перевірка:

```bash
kubectl get pods
kubectl logs deployment/course-app-course-app

NAME                                     READY   STATUS    RESTARTS      AGE
course-app-course-app-6d8c98bf4f-c2q47   1/1     Running   0             39s
course-app-course-app-6d8c98bf4f-sqb66   1/1     Running   0             49s
course-app-course-app-6d8c98bf4f-wcbgj   1/1     Running   0             60s
dragonfly-0                              1/1     Running   0             5m9s
redis-master-0                           1/1     Running   1 (12m ago)   3d22h
Found 3 pods, using pod/course-app-course-app-6d8c98bf4f-wcbgj
INFO:     Started server process [1]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8080 (Press CTRL+C to quit)
INFO:     10.244.0.1:51796 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:51806 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:44714 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:44716 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:44734 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:58506 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:58508 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:58518 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:57926 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:57936 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:57944 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:51568 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:51572 - "GET /healthz HTTP/1.1" 200 OK
INFO:     10.244.0.1:51574 - "GET /healthz HTTP/1.1" 200 OK
```

---

## 7. Створення RBAC

Було створено файл:

```text
rbac-dragonfly-viewer.yaml
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: db-viewer

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: db-readonly
rules:
  - apiGroups:
      - dragonflydb.io
    resources:
      - dragonflies
    verbs:
      - get
      - list
      - watch

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: db-viewer-readonly
subjects:
  - kind: ServiceAccount
    name: db-viewer
    namespace: default
roleRef:
  kind: Role
  name: db-readonly
  apiGroup: rbac.authorization.k8s.io
```

Застосування:

```bash
kubectl apply -f apps/course-app/k8s/rbac-dragonfly-viewer.yaml
serviceaccount/db-viewer created
role.rbac.authorization.k8s.io/db-readonly created
rolebinding.rbac.authorization.k8s.io/db-viewer-readonly created
```

---

## 8. Перевірка RBAC

Перевірка доступу на читання:

```bash
kubectl auth can-i list dragonflies --as=system:serviceaccount:default:db-viewer
yes
```

Результат:

```bash
yes
```

Перевірка заборони на видалення:

```bash
kubectl auth can-i delete dragonflies --as=system:serviceaccount:default:db-viewer
no
```

Результат:

```bash
no
```

---

## 9. Висновок

У межах домашнього завдання було:

- встановлено Dragonfly Operator
- перевірено появу CRD `dragonflies`
- створено Dragonfly instance через Custom Resource
- перевірено роботу Dragonfly через `redis-cli`
- інтегровано `course-app` з Dragonfly
- створено ServiceAccount `db-viewer`
- створено Role `db-readonly`
- створено RoleBinding
- перевірено RBAC через `kubectl auth can-i`
