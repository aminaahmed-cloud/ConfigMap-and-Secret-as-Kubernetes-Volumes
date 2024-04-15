# ConfigMap and Secret as Kubernetes Volumes

This guide provides step-by-step instructions on how to use ConfigMap and Secret volumes in Kubernetes pods. I'll cover when to use ConfigMap and Secret volumes, how to create and manage them, and how to pass configuration files to Kubernetes pods.

## When to use ConfigMap and Secret Volumes?

ConfigMap volumes are ideal for storing configuration files, environment variables, or any non-sensitive configuration data. Secret volumes, on the other hand, are used to store sensitive information such as passwords, API keys, or certificates.

## Configuration Files usages in Pods

Configuration files are essential for defining the behavior of applications running in pods. By using ConfigMap volumes, you can inject configuration files directly into pods without modifying the application's container image. Secret volumes provide a secure way to pass sensitive configuration data to pods.

## How to pass these config files to Kubernetes pods?

We'll demonstrate how to pass configuration files to Kubernetes pods using a Mosquitto deployment as an example.

### 1. Mosquitto Pod without any Volumes

- Create a Mosquitto deployment YAML file without any volumes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
  labels:
    app: mosquitto
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
      containers:
      - name: mosquitto
        image: eclipse-mosquitto:2.0
        ports:
        - containerPort: 1883
```

- Apply the YAML file to create the Mosquitto pod:

```sh
kubectl apply -f mosquitto-without-volumes.yaml
```

<img src="https://i.imgur.com/bhruKdD.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

### 2. Overwrite the mosquitto.conf file

**- Overwrite the mosquitto.conf file**

- Create a ConfigMap containing the mosquitto.conf file:

```yaml

apiVersion: v1
kind: ConfigMap
metadata:
    name: mosquitto-config-file
data:
    mosquitto.conf: |
        log_dest stdout
        log_type all
        log_timestamp true
        listener 9001
```

- Create a Secret containing the secret.file:

```yaml

apiVersion: v1
kind: Secret
metadata:
    name: mosquitto-secret-file
type: Opaque
data:
    secret.file: |
        VGVjaFdvcmxkMjAyMyEgLW4K
```

<img src="https://i.imgur.com/xNNIBAI.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

---

### 3. Create Mosquitto Deployment File with Volumes

- Create a Mosquitto deployment YAML file with ConfigMap and Secret volumes:

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
  labels:
    app: mosquitto
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
        containers:
          - name: mosquitto
            image: eclipse-mosquitto:2.0
            ports:
              - containerPort: 1883
            volumeMounts: 
              - name: mosquitto-config
                mountPath: /mosquitto/config
              - name: mosquitto-secret
                mountPath: /mosquitto/secret
                readOnly: true
        volumes:
          - name: mosquitto-config
            configMap:
              name: mosquitto-config-file
          - name: mosquitto-secret
            secret:
              secretName: mosquitto-secret-file
```

- Apply the YAML file to create the Mosquitto pod with volumes:

```bash
kubectl apply -f mosquitto.yaml
```

<img src="https://i.imgur.com/fF1jX58.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
