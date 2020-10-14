# Quarkus-orm-postgres-kubernates-istio
Sample quarkus Hibernate orm application which connects postgress which will be running within kubernetes pod. And ISTIO Envoy side car injected along with our container.

## Proof of Concepts:
~~~
1) Install Istio and Inject Envoy Sidecar for default namespace
2) Run Postgres within Kubernates with volumes
3) Build and push the application(Quarkus Hibernate ORM) image into GCR using Quarkus Kubernetes Extention
4) Deploy the image from GCR into GKE
5) Generate some traffic and Monitor Kiali dashboard 
~~~

# Steps:

## 1) Install ISTIO within Kubernetes Cluster and inject Envoy sidecar for 'default' namespace
~~~
connect to Kuberneters Cluster

curl -L https://istio.io/downloadIstio | sh -

cd istio-1.7.3 (this may change)

export PATH=$PWD/bin:$PATH

istioctl install --set profile=demo

kubectl label namespace default istio-injection=enabled

kubectl apply -f samples/addons
while ! kubectl wait --for=condition=available --timeout=600s deployment/kiali -n istio-system; do sleep 1; done

istioctl dashboard kiali

~~~

## 2) Download the code and deploy Postgress Resources
~~~
git clone https://github.com/mosesalphonse/Quarkus-orm-postgres-kubernates-istio.git

cd Quarkus-orm-postgres-kubernates-istio

kubectl create -f postgres/yamls/postgres.yaml

~~~

## 3) Build and push the application(Quarkus Hibernate ORM) image into GCR using Quarkus Kubernetes Extention
~~~
cd qaurkus-orm

JVM:

mvn clean package -Dmaven.test.skip=true -Dquarkus.container-image.push=true

Native:

mvn package -Dmaven.test.skip=true -Pnative -Dquarkus.native.container-build=true -Dquarkus.container-image.push=true

Note: If the above builds are successfull, you may find the image in the GCR
~~~

## 4) Deploy the image within GKE using the console
~~~
Go to 'Worklods' of GKE

Select 'Deploy' option

Select 'Existing Container Image' opion and select the appropriate image

Click 'Continue' 

provide application name as 'Quarkus-hibernate' and leave the namespace to 'default'

And Click Deploy

Wait till deploying and then expose the service as 'Quarkus-hibernate-service', target port should be 8080

~~~

## 5) Generate some traffic and Monitor Kiali dashboard

~~~

Generate some http traffic using CURL in a seperate terminal as below;

for i in `seq 1 2000`; do curl http://34.71.232.241/; done # change the IP addres of your ingres controller

for i in `seq 1 2000`; do curl http://34.121.217.20/fruits; done # change the IP addres of your ingres controller

Access Kiali dashboard (makesure ISTIO on your PATH where you execute the below commend)

istioctl dashboard kiali

You may monitor the the application owithin kiali dashboard.

~~~

Note: This code was tested in GKE on a specific Kubernates and ISTIO versions, this may not work in the different versions of the software.
