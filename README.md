
# PySpark &amp; Jupyter Notebooks Deployed On Kubernetes

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/logo.png?raw=true)

Install Spark On Kubernetes via **helm chart**

The control-plane & worker nodes addresses are :
```
192.168.56.115
192.168.56.116
192.168.56.117
```
![alt text](https://raw.githubusercontent.com/kayvansol/Ingress/main/pics/vmnet.png?raw=true)

Kubernetes cluster **nodes** :

![alt text](https://raw.githubusercontent.com/kayvansol/Ingress/main/pics/nodes.png?raw=true)

you can install helm via the link [helm](https://helm.sh/docs/intro/install) :

***

The Steps :

you can install helm chart via the link [helm chart](https://artifacthub.io/packages/helm/bitnami/spark/8.7.2?modal=install) :

**Important:** the spark version of helm chart must be the same as the PySpark version of jupyter

1) Install spark via helm chart **(bitnami)** :

<img src="https://raw.githubusercontent.com/kayvansol/SparkOnKubernetes/main/img/bitnami.png" width="500" height="200">
   
```
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm search repo bitnami
$ helm install kayvan-release bitnami/spark --version 8.7.2
```

2) Deploy Jupyter workloads :

jupyter.yaml :
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
   name: jupiter-spark
   namespace: default
spec:
   replicas: 1
   selector:
      matchLabels:
         app: spark
   template:
      metadata:
         labels:
            app: spark
      spec:
         containers:
            - name: jupiter-spark-container
              image: docker.arvancloud.ir/jupyter/all-spark-notebook
              imagePullPolicy: IfNotPresent
              ports:
              - containerPort: 8888
              env: 
              - name: JUPYTER_ENABLE_LAB
                value: "yes"
---
apiVersion: v1
kind: Service
metadata:
   name: jupiter-spark-svc
   namespace: default
spec:
   type: NodePort
   selector:
      app: spark
   ports:
      - port: 8888
        targetPort: 8888
        nodePort: 30001
---
apiVersion: v1
kind: Service
metadata:
  name: jupiter-spark-driver-headless
spec:
  clusterIP: None
  selector:
    app: spark
```

```
kubectl apply -f jupyter.yaml
```

the installed **pods** :

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter0.png?raw=true)

and **Services** (headless for statefull) :

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter1.png?raw=true)

Note: **spark master url** address is : 
```
spark://kayvan-release-spark-master-0.kayvan-release-spark-headless.default.svc.cluster.local:7077
```

3) Open jupyter notebook and write some python codes based on pyspark and press **shift + enter** keys on each block to execute:
```python

#import os

#os.environ['PYSPARK_SUBMIT_ARGS']='pyspark-shell'
#os.environ['PYSPARK_PYTHON']='/opt/bitnami/python/bin/python'
#os.environ['PYSPARK_DRIVER_PYTHON']='/opt/bitnami/python/bin/python'

from pyspark.sql import SparkSession

spark = SparkSession.builder.master("spark://kayvan-release-spark-master-0.kayvan-release-spark-headless.default.svc.cluster.local:7077")\
            .appName("Mahla").config('spark.driver.host', socket.gethostbyname(socket.gethostname()))\
            .getOrCreate()

```

note: 
```
socket.gethostbyname(socket.gethostname()) ---> returns the jupyter pod's ip address
```

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter.png?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter2.png?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter3.png?raw=true)

enjoying from sending python codes to spark cluster on kubernetes via jupyter.

Note: of course you can work with pyspark single node installed on jupyter without kubernetes and when you will be sure that the code is correct, then send it via spark-submit or like above code to spark cluster on kubernetes.

***

# Deploy on the Docker Desktop :

docker-compose.yml :

```yaml
version: '3.6'

services:

  spark-master:
    container_name: spark
    image: docker.arvancloud.ir/bitnami/spark:3.5.0
    environment:
      - SPARK_MODE=master
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
      - SPARK_USER=root   
      - PYSPARK_PYTHON=/opt/bitnami/python/bin/python3
    ports:
      - 127.0.0.1:8081:8080
      - 127.0.0.1:7077:7077
    networks:
      - spark-network
    

  spark-worker:
    image: docker.arvancloud.ir/bitnami/spark:3.5.0
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark:7077
      - SPARK_WORKER_MEMORY=2G
      - SPARK_WORKER_CORES=2
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
      - SPARK_USER=root
      - PYSPARK_PYTHON=/opt/bitnami/python/bin/python3
    networks:
      - spark-network


  jupyter:
    image:  docker.arvancloud.ir/jupyter/all-spark-notebook:latest
    container_name: jupyter
    ports:
      - "8888:8888"
    environment:
      - JUPYTER_ENABLE_LAB=yes
    networks:
      - spark-network
    depends_on:
      - spark-master


networks:
  spark-network:
```

run in cmd :
```bash
docker-compose up --scale spark-worker=2
```

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter4.png?raw=true)

Copy csv file to inside spark worker container :
```bash
docker cp file.csv spark-worker-1:/opt/file
docker cp file.csv spark-worker-2:/opt/file
```

Open jupyter notebook and write some python codes based on pyspark and press **shift + enter** keys on each block to execute:

```python
from pyspark.sql import SparkSession

# Create a Spark session
spark = SparkSession.builder.appName("YourAppName").master("spark://8fa1bd982ade:7077").getOrCreate()

```

```python
data = spark.read.csv("/opt/file/file.csv", header=True)
data.limit(3).show()
```

```python
spark.stop()
```

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter5.png?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter6.png?raw=true)

Note again: you can work with pyspark single node installed on jupyter without spark cluster and when you will be sure that the code is correct, then send it via spark-submit or like above code to spark cluster on docker desktop.

***

# Execute on the pyspark that installed on jupyter, no on spark cluster

Copy csv file to inside jupyter container :
```bash
docker cp file.csv jupyter:/opt/file
```

```python
from pyspark.sql import SparkSession

# Create a Spark session
spark = SparkSession.builder.appName("YourAppName").getOrCreate()

data = spark.read.csv("/opt/file/file.csv", header=True)
data.limit(3).show()

spark.stop()

```

and also you can practice on single node pyspark in jupyter :

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter7.png?raw=true)

