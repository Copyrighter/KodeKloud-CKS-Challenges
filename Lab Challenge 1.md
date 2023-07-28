# KodeKloud-CKS-Challenges
Answers for KodeKloud CKS Challenges

Answer

docker image ls

docker images | awk '/nginx/ {print $1 ":" $2}' | sort -u

# scan automatically in nginx images in repository.
for i in $(docker images | awk '/nginx/ {print $1 ":" $2}' | sort -u) ; do echo $i ; trivy image --severity CRITICAL $i | grep Total ; done

trivy image nginx:alpine
trivy image nginx:alpine --severity CRITICAL | grep Total

systemctl status apparmor
cat /sys/module/apparmor/parameters/enabled
aa-status
cp /root/usr.sbin.nginx /etc/apparmor.d/usr.sbin.nginx
apparmor_parser /etc/apparmor.d/usr.sbin.nginx
cat /sys/kernel/security/apparmor/profiles | grep custom-nginx

vi 1.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: alpha-xyz
  name: alpha-xyz
  namespace: alpha
spec:
  replicas: 1
  selector:
    matchLabels:
      app: alpha-xyz
  strategy: {}
  template:
    metadata:
      annotations:
        container.apparmor.security.beta.kubernetes.io/nginx: localhost/custom-nginx
      creationTimestamp: null
      labels:
        app: alpha-xyz
    spec:
      volumes:
       - name: data-volume
         persistentVolumeClaim:
           claimName: alpha-pvc
      containers:
      - image: nginx:alpine
        name: nginx
        volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: data-volume
        ports:
        - containerPort: 80
---		  
apiVersion: v1
kind: Service
metadata:
  labels:
    app: alpha-xyz
  name: alpha-svc
  namespace: alpha
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: alpha-xyz
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-inbound
  namespace: alpha
spec:
  podSelector:
    matchLabels:
      app: alpha-xyz
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: middleware
      ports:
       - protocol: TCP
         port: 80
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: alpha-pvc
  namespace: alpha
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
  volumeMode: Filesystem
---

kubectl delete --force -f 1.yaml

kubectl create -f 1.yaml

kubectl exec -n alpha -it middleware -- nc -zvw 5 alpha-svc.alpha.svc.cluster.local 80
kubectl -n alpha exec alpha-xyz-69b5b4b999-ztts7 -c custom-nginx -it -- curl alpha-svc.alpha.svc.cluster.local
curl alpha-svc.alpha.svc.cluster.local

nslookup 10-50-0-6.alpha.pod.cluster.local
curl 10-50-0-6.alpha.pod.cluster.local

kubectl -n alpha exec alpha-xyz-69b5b4b999-ztts7 -c custom-nginx -it -- sh
kubectl -n alpha exec middleware -it -- nc -zvw 5 alpha-svc.alpha.svc.cluster.local 80
