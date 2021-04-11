# Java, Docker e Kubernetes


## Rodando sua aplicação Java no Kubernetes. Do deploy ao debug sem medo!

Neste projeto foi criado o desafio de construir um ambiente Kubernetes local para que possamos aprender a tecnologia, mostrando como 
mover uma aplicação java/spring boot para o Docker e Kubernetes, sem medo de errar. 
Foram criados os recursos necessários para fazer o deploy no cluster e configurar a aplicação a fim de fazer o debug enquanto ela está 
rodando no Kubernetes.



## Parte 1 - Aplicativo base:

### Requerimentos:

**Docker e Make (Opcional)**

**Java 14**

Ajuda para instalar as ferramentas:

https://github.com/sandrogiacom/k8s

### Build e execução da aplicação:

Spring boot e banco de dados mysql executando no docker

**Clone do repositório**
```bash
git clone https://github.com/sandrogiacom/java-kubernetes.git
```

**Build da aplicação**
```bash
cd java-kubernetes
mvn clean install
```

**Inicializar o banco de dados**
```bash
make run-db
```

**Executar a aplicação**
```bash
java --enable-preview -jar target/java-kubernetes.jar
```

**Verificação**

http://localhost:8080/app/users

http://localhost:8080/app/hello

## Parte 2 - Aplicativo no Docker:

Criando um Dockerfile:

```yaml
FROM openjdk:14-alpine
RUN mkdir /usr/myapp
COPY target/java-kubernetes.jar /usr/myapp/app.jar
WORKDIR /usr/myapp
EXPOSE 8080
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar app.jar" ]
```

**Build da aplicação e imagem do docker**

```bash
make build
```

Criando e executando o banco de dados
```bash
make run-db
```

Criando e executando a aplicação
```bash
make run-app
```

**Verificação**

http://localhost:8080/app/users

http://localhost:8080/app/hello

Parar tudo(banco de dados e aplicação):

```
docker stop mysql57 myapp
```

## Parte 3 - Aplicativo no Kubernetes:

Temos um aplicativo e uma imagem em execução no docker
Agora, implantamos o aplicativo em um cluster Kubernetes em execução em nossa máquina

Preparação

### Inicializar minikube
```
make k-setup
``` 
inicializa minikube, habilita o ingress e cria o namespace dev-to

### Verificar IP

```
minikube -p dev.to ip
```

### Minikube dashboard

```
minikube -p dev.to dashboard
```

### Implantação do banco de dados

Criação e implantação do mysql e do serviço

```
make k-deploy-db
```

```
kubectl get pods -n dev-to
```

```
kubectl logs -n dev-to -f <pod_name>
```

```
kubectl port-forward -n dev-to <pod_name> 3306:3306
```

## Build da aplicação e implantação

build app

```
make k-build-app
```

Criar a imagem do docker dentro da instância do minikube:

```
make k-build-image
```

OU

```
minikube cache add java-k8s
```  


criação e implantação do serviço do aplicativo:

```
make k-deploy-app
``` 

**Verificação**

```
kubectl get services -n dev-to
```

Para acessar o aplicativo:

```
minikube -p dev.to service -n dev-to myapp --url
```

http://172.17.0.5:32594/app/users
http://172.17.0.5:32594/app/hello

## Verificação os pods

```
kubectl get pods -n dev-to
```

```
kubectl -n dev-to logs myapp-6ccb69fcbc-rqkpx
```

## Mapear para dev.local

obter o IP do minikube
```
minikube -p dev.to ip
``` 

Editar `hosts` 
```
sudo vim /etc/hosts
```

Replicas
```
kubectl get rs -n dev-to
```

Obter e Deletar pod
```
kubectl get pods -n dev-to
```

```
kubectl delete pod -n dev-to myapp-f6774f497-82w4r
```

Escalar(Scale)
```
kubectl -n dev-to scale deployment/myapp --replicas=2
```

Testar replicas
```
while true
do curl "http://dev.local/app/hello"
echo
sleep 2
done
```

Testar replicas com wait
```
while true
do curl "http://dev.local/app/wait"
echo
done
```

## Verificar a url do aplicativo
```
minikube -p dev.to service -n dev-to myapp --url
```

Troque seu IP e PORT conforme sua necessidade
```
curl -X GET http://dev.local/app/users
```

Adicionar novo Usuário
```
curl --location --request POST 'http://dev.local/app/users' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "new user",
    "birthDate": "2010-10-01"
}'
```

## Parte 4 - debug do aplicativo:

Adicionar  JAVA_OPTS: "-agentlib:jdwp=transport=dt_socket,address=*:5005,server=y,suspend=n -Xms256m -Xmx512m -XX:MaxMetaspaceSize=128m"
Mudificar CMD para ENTRYPOINT no Dockerfile

```
kubectl get pods -n=dev-to
```

```
kubectl port-forward -n=dev-to <pod_name> 5005:5005
```

## KubeNs e Stern

```
kubens dev-to
```

```
stern myapp
```

## Inicializar tudo

```
make k:all
```

## Referências

https://dev.to/sandrogiacom/kubernetes-for-java-developers-setup-41nk

https://kubernetes.io/docs/home/

https://minikube.sigs.k8s.io/docs/
