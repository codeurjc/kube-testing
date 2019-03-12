# Kubernetes Chaos Demo materials

Entrada:

https://blog.loadmill.com/run-chaos-experiments-without-risking-your-job-2c8a5f4b0bfc

Este trabajo consiste en desplegar una app y un pod-chaos-monkey que es una implementación simple de chaos-monkey.

Este chaos-monkey elimina un pod cada 30 segundos (configurable) y con la ayuda de LoadMills hacemos peticiones y vemos como se comporta la aplicación.

# Como usar este ejemplo

1. Necesitamos activar el ingress controller del cluster. Para esto usamos Helm escribiendo estos comandos:

1.1 Instalar el ingress controller

```
$ kubectl create serviceaccount --namespace kube-system tiller
$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
$ helm init --service-account tiller
$ helm install --name nginx-ingress stable/nginx-ingress
```

1.2 Instalar la aplicación el token de LoadMill como secreto

Partimos que tenemos el token en el fichero `loadmill-token` que hemos creado así:

`$ echo -n '8dffd41bd4...71e0bb0a5589' > loadmill-token`

Creamos el secreto

`$ kubectl create secret generic loadmill-token --from-file=./loadmill-token`

1.3 Desplegamos la aplicación

```
$ kubectl create -f deployment.yaml
$ kubectl create -f service.yaml 
$ kubectl create -f basic-ingress.yaml 
```

Accedemos a la aplicación conociendo la IP (OJO: puede tardar hasta 10 minutos en estar disponible)

`$ kubectl get ing`

```
NAME            HOSTS   ADDRESS        PORTS   AGE
basic-ingress   *       34.95.96.252   80      19m
```

luego la URL de la aplicación es http://34.95.96.252

Con esta url configuramos el test de carga en LoadMill y vemos que no hay errores.

Añadimos chaos, vamos a desplegar pod-chaos-monkey que eliminará un pod de la APP cada 10 segundos:

```
$ kubectl create -f chaos-rbac.yaml -f kube-chaos.yaml`