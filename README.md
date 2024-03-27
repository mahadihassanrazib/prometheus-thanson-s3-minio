kubectl apply -f monitoring-ns.yaml

kubectl create -f prometheus-operator-crds/

kubectl apply -R -f prometheus-operator

kubectl get po -n monitoring --show-labels

app.kubernetes.io/name=prometheus-operator

kubectl logs -l app.kubernetes.io/name=prometheus-operator -n monitoring -f

vagrant ssh -- -L 7001:localhost:5432

minikube addons enable storage-provisioner

minikube addons enable volumesnapshots

minikube addons enable csi-hostpath-driver

kubectl apply -f prometheus

kubectl logs -l app.kubernetes.io/name=prometheus -n monitoring -f

# minio install
kubectl apply -f minio.ns.yaml

kubectl apply minio/

kubectl get svc -n minio

minikube service minio-console --url -n minio




=========================================================
kubectl apply -f monitoring-ns.yaml

helm repo add bitnami https://charts.bitnami.com/bitnami

helm repo update

# include --set prometheus.additionalArgs=["--storage.tsdb.max-block-duration=10m","--storage.tsdb.min-block-duration=10m"]

helm install --namespace monitoring kube-prometheus \
  --set prometheus.thanos.create=true \
  --set operator.service.type=ClusterIP \
  --set prometheus.service.type=ClusterIP \
  --set alertmanager.service.type=ClusterIP \
  --set prometheus.thanos.service.type=ClusterIP \
  --set prometheus.externalLabels.cluster="data-producer-0" \
  bitnami/kube-prometheus

========== logs ===============
NAME: kube-prometheus
LAST DEPLOYED: Mon Mar 25 11:49:02 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kube-prometheus
CHART VERSION: 8.30.1
APP VERSION: 0.72.0

** Please be patient while the chart is being deployed **

Watch the Prometheus Operator Deployment status using the command:

    kubectl get deploy -w --namespace monitoring -l app.kubernetes.io/name=kube-prometheus-operator,app.kubernetes.io/instance=kube-prometheus

Watch the Prometheus StatefulSet status using the command:

    kubectl get sts -w --namespace monitoring -l app.kubernetes.io/name=kube-prometheus-prometheus,app.kubernetes.io/instance=kube-prometheus

Prometheus can be accessed via port "9090" on the following DNS name from within your cluster:

    kube-prometheus-prometheus.monitoring.svc.cluster.local

To access Prometheus from outside the cluster execute the following commands:

    echo "Prometheus URL: http://127.0.0.1:9090/"
    kubectl port-forward --namespace monitoring svc/kube-prometheus-prometheus 9090:9090

Thanos Sidecar can be accessed via port "10901" on the following DNS name from within your cluster:

    kube-prometheus-prometheus-thanos.monitoring.svc.cluster.local

Watch the Alertmanager StatefulSet status using the command:

    kubectl get sts -w --namespace monitoring -l app.kubernetes.io/name=kube-prometheus-alertmanager,app.kubernetes.io/instance=kube-prometheus

Alertmanager can be accessed via port "9093" on the following DNS name from within your cluster:

    kube-prometheus-alertmanager.monitoring.svc.cluster.local

To access Alertmanager from outside the cluster execute the following commands:

    echo "Alertmanager URL: http://127.0.0.1:9093/"
    kubectl port-forward --namespace monitoring svc/kube-prometheus-alertmanager 9093:9093

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - alertmanager.resources
  - blackboxExporter.resources
  - operator.resources
  - prometheus.resources
  - prometheus.thanos.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

============== logs end ===================

# kubectl get svc | grep prometheus-operator-kube-p-prometheus-thanos

KEY: ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789


helm install --namespace monitoring thanos bitnami/thanos --values values.yaml

============ Thanos installation logs ======================
NAME: thanos
LAST DEPLOYED: Mon Mar 25 12:24:08 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: thanos
CHART VERSION: 14.0.2
APP VERSION: 0.34.1

** Please be patient while the chart is being deployed **

Thanos chart was deployed enabling the following components:
- Thanos Query
- Thanos Bucket Web
- Thanos Compactor
- Thanos Ruler
- Thanos Store Gateway

Thanos Query can be accessed through following DNS name from within your cluster:

    thanos-query.monitoring.svc.cluster.local (port 9090)

To access Thanos Query from outside the cluster execute the following commands:

1. Get the Thanos Query URL by running these commands:

    export SERVICE_PORT=$(kubectl get --namespace monitoring -o jsonpath="{.spec.ports[0].port}" services thanos-query)
    kubectl port-forward --namespace monitoring svc/thanos-query ${SERVICE_PORT}:${SERVICE_PORT} &
    echo "http://127.0.0.1:${SERVICE_PORT}"

2. Open a browser and access Thanos Query using the obtained URL.

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - bucketweb.resources
  - compactor.resources
  - query.resources
  - queryFrontend.resources
  - ruler.resources
  - storegateway.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

================= Tahnos installation logs End ======================


# install Grafana

helm install --namespace monitoring grafana bitnami/grafana \
  --set service.type=ClusterIP \
  --set admin.password=admin


================== Grafana logs =================
NAME: grafana
LAST DEPLOYED: Mon Mar 25 12:36:55 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: grafana
CHART VERSION: 10.0.3
APP VERSION: 10.4.1

** Please be patient while the chart is being deployed **

1. Get the application URL by running these commands:
    echo "Browse to http://127.0.0.1:8080"
    kubectl port-forward svc/grafana 8080:3000 &

2. Get the admin credentials:

    echo "User: admin"
    echo "Password: $(kubectl get secret grafana-admin --namespace monitoring -o jsonpath="{.data.GF_SECURITY_ADMIN_PASSWORD}" | base64 -d)"
# Note: Do not include grafana.validateValues.database here. See https://github.com/bitnami/charts/issues/20629


WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - grafana.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

============ Grafana logs End ================




# install mariaDB

helm install --namespace monitoring mariadb \
  --set rootUser.password=admin \
  --set replication.password=admin \
  --set db.user=admin \
  --set db.password=admin \
  --set db.name=mythanos \
  --set slave.replicas=1 \
  --set metrics.enabled=true \
  --set metrics.serviceMonitor.enabled=true \
  bitnami/mariadb

================= mariaDB logs ==============
NAME: mariadb
LAST DEPLOYED: Mon Mar 25 12:49:26 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: mariadb
CHART VERSION: 17.0.1
APP VERSION: 11.2.3

** Please be patient while the chart is being deployed **

Tip:

  Watch the deployment status using the command: kubectl get pods -w --namespace monitoring -l app.kubernetes.io/instance=mariadb

Services:

  echo Primary: mariadb.monitoring.svc.cluster.local:3306

Administrator credentials:

  Username: root
  Password : $(kubectl get secret --namespace monitoring mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 -d)

To connect to your database:

  1. Run a pod that you can use as a client:

      kubectl run mariadb-client --rm --tty -i --restart='Never' --image  docker.io/bitnami/mariadb:11.2.3-debian-12-r4 --namespace monitoring --command -- bash

  2. To connect to primary service (read/write):

      mysql -h mariadb.monitoring.svc.cluster.local -uroot -p my_database

To upgrade this helm chart:

  1. Obtain the password as described on the 'Administrator credentials' section and set the 'auth.rootPassword' parameter as shown below:

      ROOT_PASSWORD=$(kubectl get secret --namespace monitoring mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 -d)
      helm upgrade --namespace monitoring mariadb oci://registry-1.docker.io/bitnamicharts/mariadb --set auth.rootPassword=$ROOT_PASSWORD

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - metrics.resources
  - primary.resources
  - secondary.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
===================== End ======================























  