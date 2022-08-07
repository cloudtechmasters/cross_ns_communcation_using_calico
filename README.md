We are going to establish cross namepsace communication across pods using calico and network policy in this tutorial.

Note: Assuming you have EKS cluster already running.


Steps:

1. Install postgress and springboot app
2. Install calico using helm
3. Update network policy to default deny  
4. Update network policy to allow communication from springboot to postgress



1. Install postgress and springboot app


# k apply -f postgress/ -n backend
namespace/backend created
configmap/postgres-db-config created
statefulset.apps/postgresql-db created
service/postgres-db created

# kubectl create configmap hostname-config --from-literal=postgres_host=$(kubectl get svc postgres-db -n backend -o jsonpath="{.spec.clusterIP}") -n frontend

# k apply -f springboot/ -n frontend
namespace/frontend unchanged
secret/postgres-secrets unchanged
deployment.apps/spring-boot-postgres-sample created
service/spring-boot-postgres-sample unchanged


2. Install calico using helm

# kubectl create namespace tigera-operator
namespace/tigera-operator created
[root@ip-172-31-3-61 opt]# helm install calico projectcalico/tigera-operator --version v3.23.3 --namespace tigera-operator
W0807 03:47:15.421088    3852 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
W0807 03:47:15.679160    3852 warnings.go:70] policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
NAME: calico
LAST DEPLOYED: Sun Aug  7 03:47:13 2022
NAMESPACE: tigera-operator
STATUS: deployed
REVISION: 1
TEST SUITE: None


3. Update network policy to default deny  
4. Update network policy to allow communication from springboot to postgress
