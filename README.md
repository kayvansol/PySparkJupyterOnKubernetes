
# PySpark &amp; Jupyter Notebooks Deployed OnKubernetes

Spark On Kubernetes via **helm chart**

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

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter.png?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter2.png?raw=true)

![alt text](https://raw.githubusercontent.com/kayvansol/PySparkJupyterOnKubernetes/main/img/PysparkJupyter3.png?raw=true)

enjoying from sending python codes to spark cluster on kubernetes via jupyter.

Note: of course you can work with pyspark single node installed on jupyter without kubernetes and when you will be sure that the code is correct, then send it via spark-submit or like above code to spark cluster on kubernetes.
