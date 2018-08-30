# lstm
lstm tensorflow python flask docker kubernetes


sudo docker build -t lstm:latest .
sudo docker images | grep lstm
sudo docker tag xxx dagilgon/lstm:latest
sudo docker login
sudo docker push dagilgon/lstm:latest


#cat del token de auth
sudo cat ~/.docker/config.json
{
	"auths": {
		"https://index.docker.io/v1/": {
			"auth": "xxx"
		}
	},
	"HttpHeaders": {
		"User-Agent": "Docker-Client/17.12.1-ce (linux)"
	}
}

# creamos secreto del login
kubectl create secret docker-registry regcred --docker-server=xxx --docker-username=login --docker-password=xxx --docker-email=login@mail.com
# ver secreto
kubectl get secret regcred --output=yaml
# verlo mejor
kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode

pod -> pod.yaml
-----------------------------------
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: dagilgon/lstm
  imagePullSecrets:
  - name: regcred
------------------------------------
service -> service.yaml
------------------------------------
apiVersion: v1
kind: Service
metadata:
  name: lstm-lb
spec:
  type: LoadBalancer
  ports:
  - port: 5000
    protocol: TCP
    targetPort: 5000
  selector:
    app: lstm
--------------------------------------
deployment -> deployment.yaml
--------------------------------------
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: lstm
spec:
  replicas: 3
  minReadySeconds: 60
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: lstm
    spec:
      containers:
        - image: dagilgon/lstm
          imagePullPolicy: Always
          name: lstm
          ports:
            - containerPort: 5000

apiVersion: apps/v1 
kind: Deployment
metadata:
  name: lstm-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4 # Update the replicas from 2 to 4
  template:
    metadata:
      labels:
        app: lstm
    spec:
      containers:
      - name: lstm
        image: dagilgon/lstm
        ports:
        - containerPort: 5000


#pod kmaster
kubectl create -f pod.yaml
kubectl get pod lstm

#pod local
kubectl --kubeconfig ~/kubernetes/admin.conf create -f pod.yaml
kubectl --kubeconfig ~/kubernetes/admin.conf  get pod lstm

#service local
kubectl --kubeconfig ~/kubernetes/admin.conf create -f service.yaml

#deployment local
kubectl --kubeconfig ~/kubernetes/admin.conf apply -f sa-frontend-deployment.yaml
