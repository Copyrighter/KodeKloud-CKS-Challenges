# KodeKloud-CKS-Challenges
Answers for KodeKloud CKS Challenge 1

Answer
============

docker image ls<br />

docker images | awk '/nginx/ {print $1 ":" $2}' | sort -u<br />

// scan automatically in nginx images in repository.<br />
for i in $(docker images | awk '/nginx/ {print $1 ":" $2}' | sort -u) ; do echo $i ; trivy image --severity CRITICAL $i | grep Total ; done<br />

trivy image nginx:alpine<br />
trivy image nginx:alpine --severity CRITICAL | grep Total<br />
	
systemctl status apparmor<br />
cat /sys/module/apparmor/parameters/enabled<br />
aa-status<br />
cp /root/usr.sbin.nginx /etc/apparmor.d/usr.sbin.nginx<br />
apparmor_parser /etc/apparmor.d/usr.sbin.nginx<br />
cat /sys/kernel/security/apparmor/profiles | grep custom-nginx<br />

vi 1.yaml<br />
`---<br />
`apiVersion: apps/v1<br />
kind: Deployment<br />
metadata:<br />
  labels:<br />
    app: alpha-xyz<br />
  name: alpha-xyz<br />
  namespace: alpha<br />
spec:<br />
  replicas: 1<br />
  selector:<br />
    matchLabels:<br />
      app: alpha-xyz<br />
  strategy: {}<br />
  template:<br />
    metadata:<br />
      annotations:<br />
        container.apparmor.security.beta.kubernetes.io/nginx: localhost/custom-nginx<br />
      creationTimestamp: null<br />
      labels:<br />
        app: alpha-xyz<br />
    spec:<br />
      volumes:<br />
       - name: data-volume<br />
         persistentVolumeClaim:<br />
           claimName: alpha-pvc<br />
      containers:<br />
      - image: nginx:alpine<br />
        name: nginx<br />
        volumeMounts:<br />
        - mountPath: "/usr/share/nginx/html"<br />
          name: data-volume<br />
        ports:<br />
        - containerPort: 80`<br />`
---<br />
apiVersion: v1<br />
kind: Service<br />
metadata:<br />
  labels:<br />
    app: alpha-xyz<br />
  name: alpha-svc<br />
  namespace: alpha<br />
spec:<br />
  ports:<br />
  - port: 80<br />
    protocol: TCP<br />
    targetPort: 80<br />
  selector:<br />
    app: alpha-xyz<br />
  sessionAffinity: None<br />
  type: ClusterIP<br />
---<br />
apiVersion: networking.k8s.io/v1<br />
kind: NetworkPolicy<br />
metadata:<br />
  name: restrict-inbound<br />
  namespace: alpha<br />
spec:<br />
  podSelector:<br />
    matchLabels:<br />
      app: alpha-xyz<br />
  policyTypes:<br />
    - Ingress<br />
  ingress:<br />
    - from:<br />
        - podSelector:<br />
            matchLabels:<br />
              app: middleware<br />
      ports:<br />
       - protocol: TCP<br />
         port: 80<br />
---<br />
apiVersion: v1<br />
kind: PersistentVolumeClaim<br />
metadata:<br />
  name: alpha-pvc<br />
  namespace: alpha<br />
spec:<br />
  accessModes:<br />
  - ReadWriteMany<br />
  resources:<br />
    requests:<br />
      storage: 1Gi<br />
  storageClassName: local-storage<br />
  volumeMode: Filesystem<br />
---<br />

kubectl delete --force -f 1.yaml<br />

kubectl create -f 1.yaml<br />

kubectl exec -n alpha -it middleware -- nc -zvw 5 alpha-svc.alpha.svc.cluster.local 80<br />
kubectl -n alpha exec alpha-xyz-69b5b4b999-ztts7 -c custom-nginx -it -- curl alpha-svc.alpha.svc.cluster.local<br />
curl alpha-svc.alpha.svc.cluster.local<br />

nslookup 10-50-0-6.alpha.pod.cluster.local<br />
curl 10-50-0-6.alpha.pod.cluster.local<br />

kubectl -n alpha exec alpha-xyz-69b5b4b999-ztts7 -c custom-nginx -it -- sh<br />
kubectl -n alpha exec middleware -it -- nc -zvw 5 alpha-svc.alpha.svc.cluster.local 80<br />
