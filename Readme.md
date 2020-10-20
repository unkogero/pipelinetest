curl -XPOST \
-H "Authorization: token xxxxx" \
-H "Accept: application/vnd.github.v3+json" \
https://api.github.com/repos/unkogero/pipelinetest/actions/workflows/3063591/dispatches \
-d '{"ref":"master","inputs": {"imagename":"bbb/ccc:ddd"}}'


curl -XPOST \
-H "Authorization: token xxxxx" \
-H "Accept: application/vnd.github.v3+json" \
https://api.github.com/repos/unkogero/pipelinetest/actions/workflows/merge.yml/dispatches \
-d '{"ref":"master","inputs": {"imagename":"bbb/ccc:ddd"}}'


=======================


# argoCD
https://www.openshift.com/blog/introduction-to-gitops-with-openshift
https://argoproj.github.io/argo-cd/getting_started/

# Create a new namespace for ArgoCD components

oc create namespace argocd

# Apply the ArgoCD Install Manifest

oc -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/v1.2.2/manifests/install.yaml
oc -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


# Get the ArgoCD Server password

ARGOCD_SERVER_PASSWORD=$(oc -n argocd get pod -l "app.kubernetes.io/name=argocd-server" -o jsonpath='{.items[*].metadata.name}')

# Patch ArgoCD Server so no TLS is configured on the server (--insecure)

PATCH='{"spec":{"template":{"spec":{"$setElementOrder/containers":[{"name":"argocd-server"}],"containers":[{"command":["argocd-server","--insecure","--staticassets","/shared/app"],"name":"argocd-server"}]}}}}'

oc -n argocd patch deployment argocd-server -p $PATCH

# Expose the ArgoCD Server using an Edge OpenShift Route so TLS is used for incoming connections

oc -n argocd create route edge argocd-server --service=argocd-server --port=http --insecure-policy=Redirect


# Download the argocd binary, place it under /usr/local/bin and give it execution permissions

curl -L https://github.com/argoproj/argo-cd/releases/download/v1.2.2/argocd-linux-amd64 -o ./argocd

chmod +x ./argocd


# Get ArgoCD Server Route Hostname

ARGOCD_ROUTE=$(oc -n argocd get route argocd-server -o jsonpath='{.spec.host}')

# Login with the current admin password

argocd --insecure --grpc-web login ${ARGOCD_ROUTE}:443 --username admin --password ${ARGOCD_SERVER_PASSWORD}

# Update admin's password

argocd --insecure --grpc-web --server ${ARGOCD_ROUTE}:443 account update-password --current-password ${ARGOCD_SERVER_PASSWORD} --new-password [pass]


https://github.com/argoproj/argocd-example-apps
ArgoCD Example Apps




#Multi
container-lab$ ./argocd cluster list
SERVER                          NAME  STATUS      MESSAGE
https://kubernetes.default.svc        Successful  

container-lab$ ./argocd cluster add
ERRO[0000] Choose a context name from:                  
CURRENT  NAME                                                                               CLUSTER                                        SERVER
*        default/c104-e-us-east-containers-cloud-ibm-com:32572/IAM#aaa@bbb.com  c104-e-us-east-containers-cloud-ibm-com:32572  https://c104-e.us-east.containers.cloud.ibm.com:32572
container-lab$ ./argocd cluster add default/c104-e-us-east-containers-cloud-ibm-com:32572/IAM#aaa@bbb.com
INFO[0000] ServiceAccount "argocd-manager" created in namespace "kube-system" 
INFO[0000] ClusterRole "argocd-manager-role" created    
INFO[0000] ClusterRoleBinding "argocd-manager-role-binding" created, bound "argocd-manager" to "argocd-manager-role" 
Cluster 'default/c104-e-us-east-containers-cloud-ibm-com:32572/IAM#aaa@bbb.com' added
container-lab$ 


https://www.openshift.com/blog/multi-cluster-management-with-gitops
