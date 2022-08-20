# A Mono-Repository example for GitOps

## Configure ArgoCD Admin permissions

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

## Create ArgoCD Applications

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