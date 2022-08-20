# A Mono-Repository example for GitOps

## Lab Env

- 容器集群：OpenShift 4.10.24
- GitOps工具: OpenShift GitOps Operator 1.5.5 (ArgoCD v2.3.7)
- 版本控制：GitHub

## 在OpenSfhit默认的ArgoCD上发布应用

### Configure ArgoCD Admin permissions

解决Login with OpenShift后在ArgoCD上没有权限操作的问题：
1. 创建`cluster-admin-group`组，将指定用户添加到该组
2. 将`cluster-admin`权限授权给`cluster-admin-group`组
```bash
oc adm policy add-cluster-role-to-group cluster-admin cluster-admin-group
```
3. 在`openshift-gitops`项目，打开Installed Operators，打开OpenShift GitOps operator
4. 打开ArgoCD，打开`openshift-gitops`, 编辑YAML，在`rbac.policy`下添加：
```yaml
g, cluster-admin-group, role:admin
```
5. 检查`argocd-rbac-cm` ConfigMap，确保`policy.csv`的值符合预期
6. 在ArgoCD上Log out再重新登录。

### Create ArgoCD Applications

Create ArgoCD applications on ArgoCD web console, or via below commands.

Create `cluster-config` application to create namespaces and rolebindings:

```bash
oc apply -f argocd/apps/cluster-config.yaml -n openshift-gitops
```

Sync `cluster-config` application mannualy on ArgoCD web console.

Create other applications:

```bash
# parksmap
oc apply -f argocd/apps/parksmap-dev.yaml -n openshift-gitops
oc apply -f argocd/apps/parksmap-test.yaml -n openshift-gitops

# spring-petclinic
oc apply -f argocd/apps/spring-petclinic-dev.yaml -n openshift-gitops
oc apply -f argocd/apps/spring-petclinic-test.yaml -n openshift-gitops
```

## 在自定义的ArgoCD上发布应用

### Create a customized ArgoCD instance

创建额外的ArgoCD实例：
1. 创建`myarogcd`项目
2. 在`myarogcd`项目，打开Installed Operators，打开OpenShift GitOps operator
3. 打开ArgoCD，创建ArgoCD，取名为`myargocd`
4. 在Networking / route中打开新的ArgoCD的地址
5. Login with OpenShift

或者运行以下命令：
```bash
oc new-project myargocd
oc apply -f argocd/argocd/argocd.yaml -n myargocd
```

### Configure ArgoCD Admin permissions

此时用户还没有ArgoCD的admin权限，所以会看不到application。

下面给用户分配ArgoCD的admin权限:
1. 创建`myargocd-admin-group`组，将指定用户添加到该组
2. 将`myargocd`项目下的`admin`权限授权给`myargocd-admin-group`组
```bash
oc adm policy add-role-to-group admin myargocd-admin-group -n myargocd
```
3. 在`myarogcd`项目，打开Installed Operators，打开OpenShift GitOps operator
4. 打开ArgoCD，打开`myargocd`, 编辑YAML，在`rbac.policy`下添加：
```yaml
g, myargocd-admin-group, role:admin
```
5. 检查`argocd-rbac-cm` ConfigMap，确保`policy.csv`的值符合预期

### Create cluster-wide resoruces
Refer to below "Clean Up" section to clean up previous created ressources if necessary.

Create cluster-wide resources in default `openshift-gitops` argocd instance:

```bash
oc apply -f argocd/apps/cluster-config.yaml -n openshift-gitops
```

### Create non cluster-wide resources
Create non cluster-wide resources in customized `myargocd` argocd instance:
```bash
# parksmap
oc label namespace parksmap-dev argocd.argoproj.io/managed-by=myargocd
oc label namespace parksmap-test argocd.argoproj.io/managed-by=myargocd

oc apply -f argocd/apps/parksmap-dev.yaml -n myargocd
oc apply -f argocd/apps/parksmap-test.yaml -n myargocd

# spring-petclinic
oc label namespace spring-petclinic-dev argocd.argoproj.io/managed-by=myargocd
oc label namespace spring-petclinic-test argocd.argoproj.io/managed-by=myargocd

oc apply -f argocd/apps/spring-petclinic-dev.yaml -n myargocd
oc apply -f argocd/apps/spring-petclinic-test.yaml -n myargocd
```

## Clean Up

Clean up priveous created applications and resources after testing.

```bash
# clean applications
oc delete application cluster-config -n openshift-gitops
oc delete application parksmap-dev -n openshift-gitops
oc delete application parksmap-test -n openshift-gitops
oc delete application spring-petclinic-dev -n openshift-gitops
oc delete application spring-petclinic-test -n openshift-gitops

# clean projects
oc delete project parksmap-dev
oc delete project parksmap-test
oc delete project spring-petclinic-dev
oc delete project spring-petclinic-test
```


## References

- [openshift-gitops-getting-started](https://github.com/redhat-developer/openshift-gitops-getting-started)
- [OpenShift GitOps docs](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.10/html/cicd/gitops#understanding-openshift-gitops)
- [ArgoCD docs](https://argo-cd.readthedocs.io/)
- [kustomize - kustomization](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/)