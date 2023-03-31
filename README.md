# helmchart
Helm chart steps for deplying Bamboo

# Deploy the database
# In this example I used postgresql database

# Step1: Add Helm repository by Bitnami
helm repo add bitnami <https://charts.bitnami.com/bitnami>
# Update Helm index charts
helm repo update

# Step2: Create a local pv definition file
$ cat local-pv.yaml
apiVersion: v1
kind: PersistentVolume # Create a PV
metadata:
  name: postgresql-data # Sets PV's name
  labels:
    type: local # Sets PV's type to local
spec:
  storageClassName: ""
  capacity:
    storage: 10Gi # Sets PV Volume
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/data/mysql" # Sets the volume's path

# Step3: Create a PVC definition file
 cat pv-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim # Create PVC
metadata:
  name: postgresql-data-claim # Sets name of PV
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteMany # Sets read and write access
  resources:
    requests:
      storage: 10Gi # Sets volume size
      
# Step4: deploy pv and pvc
kubectl apply -f local-pv.yaml
kubectl apply -f pv-claim.yaml

# Step5: Make sure that the pvc is in bound state:
$ kubectl get pv
NAME              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                           STORAGECLASS   REASON   AGE
postgresql-data   10Gi       RWX            Retain           Bound    default/postgresql-data-claim                           8m4s

$ kubectl get pvc
NAME                    STATUS   VOLUME            CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgresql-data-claim   Bound    postgresql-data   10Gi       RWX                           7m42s

# Step6: Create the psql values file
$ cat pg.yaml
# define default database user, name, and password for PostgreSQL deployment
auth:
  enablePostgresUser: true
  postgresPassword: "support1!"
  username: "bamboo"
  password: "support1!"
  database: "bamboodb"

# The postgres helm chart deployment will be using PVC postgresql-data-claim
primary:
  persistence:
    enabled: true
    existingClaim: "postgresql-data-claim"
    
# Step 7: Deploy the database
helm install postgresql-dev -f pg.yaml bitnami/postgresql

# Step 8: Confirm that the pod is running
kubectl get pod -w
