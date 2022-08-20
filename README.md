# A Mono-Repository example for GitOps

## Create ArgoCD Applications

Create ArgoCD applications on ArgoCD Web Console, or via below commands.

Create `cluster-config` application to create namespaces and rolebindings:

```bash
oc apply -f argocd/apps/cluster-config.yaml -n openshift-gitops
```

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