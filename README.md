# coder

Install coder
```bash
helm install coder coder/coder \
  -n coder \
  --set storageClassName=manual \
  --set ingress.useDefault=false \
  --set timescale.resources.limits.memory=null \
  --set timescale.resources.requests.memory=null
```

Create volume
```bash
kubectl apply -f - << EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: coder
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: "/opt/coder"
EOF
```

Run the follwoing command on the node for giving volume permission to the pod:
```bash
chown 70:70 /opt/coder
```

Setup Ingress:
```bash
kubectl apply -f - << EOF
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: coder
spec:
  tls:
    - hosts:
      - coder.k8s.shubhamtatvamasi.com
      secretName: letsencrypt
  rules:
  - host: coder.k8s.shubhamtatvamasi.com
    http:
      paths:
      - path: /proxy/
        backend:
          serviceName: envproxy
          servicePort: 8080
      - path: /
        backend:
          serviceName: cemanager
          servicePort: 8080
EOF
```

Get username and password from here:
```bash
kubectl logs deploy/cemanager | head -20
```
