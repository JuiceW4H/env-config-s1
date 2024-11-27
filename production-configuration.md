Step 1: Create namespace for production environment

kubectl create namespace production

Step 2: Create config map containing environment variables

kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --namespace=production

Step 3: Create secrets to hold env variables

kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=prodpassword123 \
  --namespace=production


Step 4: Utilize existing deployment template and modify replica amount

sed -e 's/PLACEHOLDER_NAMESPACE/production/' \
    -e 's/replicas: .*$/replicas: 3/' app-deployment-template.yaml > production-deployment.yaml


Step 5: production-deployment.yaml file if no deployment template exists

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
        env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD

Step 6: Apply deployment

kubectl apply -f production-deployment.yaml
