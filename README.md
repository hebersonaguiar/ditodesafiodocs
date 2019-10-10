Esse reposítório foi criado para servir de complementação à documetação dos projetos do Dito Desafio, que são os repositórios abaixo:

[Dito Chat Backend](https://github.com/hebersonaguiar/ditochatbackend)

[Dito Chat Frontend](https://github.com/hebersonaguiar/ditochatfrontend)

Os repositórios infomrados acima, possuem suas prórprias documentações referente ao seu uso local e em um ambiente de cluster kubernetes.

## A proposta
A proposta desse projeto é de em base ao repositório [Dito Chat](https://github.com/ditointernet/dito-chat) que foi repassado pela Dito, colocar em produção, prezando pela utilização de ferramentas de deploy contínuo, grantindo a alta disponibilidade e manutenabilidade(monitoramento e logs), para isso diversos trabalhos foram realizado, pois o repositório [Dito Chat](https://github.com/ditointernet/dito-chat) possuem apenas o código fonte da aplicação e uma breve explicação de como deve ser utilizada.

Para esse desafio foram utilizados as seguintes tecnologias:

[Docker](https://github.com/hebersonaguiar/ditodesafiodocs#docker)

[Google Cloud Plataform](https://github.com/hebersonaguiar/ditodesafiodocs#google-cloud-plataform)

[Google Kubernetes Engine](https://github.com/hebersonaguiar/ditodesafiodocs#kubernetes)

[Jenkins X](https://github.com/hebersonaguiar/ditodesafiodocs#jenkins-x)

[Helm Chart](https://github.com/hebersonaguiar/ditodesafiodocs#helm-chart)

[Chart Museum](https://github.com/hebersonaguiar/ditodesafiodocs#chart-museum)

[GitHub](https://github.com/hebersonaguiar/ditodesafiodocs)

[Skaffold](https://github.com/hebersonaguiar/ditodesafiodocs#skaffold)

[Nexus](https://github.com/hebersonaguiar/ditodesafiodocs#nexus)

[Prometheus](https://github.com/hebersonaguiar/ditodesafiodocs#prometheus)

[Grafana](https://github.com/hebersonaguiar/ditodesafiodocs#grafana)

[ELK](https://github.com/hebersonaguiar/ditodesafiodocs#elk)


Todos as tecnologias informadas acima serão descritas e detalhadas nesse documento, bem como suas integrações e uso.

## Qual ideia desse projeto?

De forma bem enxuta, a ideia desse projeto é que ao realizar alguma alteração no código da aplicação no repositório, automaticamente o deploy da aplicação para produção é acionada, mas como que isso funciona? Ao realizar o commit um webhook integrado com o Jenkins X é acionada dando inicio aó deploy, só que durante esse processo algumas ações são realizadas para que as alterações tenham efeito, que são o build da imagem da aplicação com as alterações realizadas, push da imagem para o repositório, build do helm chart com a nova versão da imagem, push do helm chart para o Chart Museum e por fim o deploy da aplicação para o cluster. 

Vale resaltar que, quando o commit é realizado, uma tag é criada e essa mesma tag segue todo o fluxo para todos os artefatos, imagem, helm chart, etc, ou seja, o rasterio da imagem dentro do fluxo é de fácil indentificação, apoiando muito na manutenção da aplicação e do ambiente.

Outro fato importante é, as informações de container como uso de memória, cpu e logs, são enviados para o Grafana e Kibana.


## Google Cloud Plataform
Google Cloud Platform também conhecida como GCP é uma suíte de cloud oferecida pelo Google, funcionando na mesma infraestrutura que a empresa usa para seus produtos dirigidos aos usuários, dentre eles o Buscador Google e o Youtube.
Para esse projeto foi utilizada uma conta pessoal. Os produtos que iremos utilizar são 

* [Google Cloud Plataform](https://github.com/hebersonaguiar/ditodesafiodocs#google-cloud-plataform), onde foi utilizado o domínio `ditochallenge.com` apontando os momes das aplicações para os serviços de entrada de requição, que pode ser ou o Service ou Ingress do Kubernetes.

* [Google Kubernetes Engine](https://github.com/hebersonaguiar/ditodesafiodocs#kubernetes), utilizado para criar o cluster do Kubernetes. Existe um ponto bem interessante que é preciso informar, ao criar o cluster Kubernetes com o Kubernetes Engine, o master é de responsabilidade da Google Cloud, ou seja, todos os servidores que estão sendo utilizados são Nodes, não precisamos nos preocupar com o master.

Existem divesas formas de criar um cluster no GCP, como por exemplo, pelo [console](https://console.cloud.google.com/kubernetes/), onde é possível escolher a quantidade de servidores, recusros e localidade, a outra forma é por linha de comando utilizando o `gcloud`, ex:

```bash
gcloud container clusters create [CLUSTER_NAME] [--zone [COMPUTE_ZONE]] -num-nodes 3
```

e a outra forma é a que utilizamos nesse projeto, utilizando o Jenkins X por linha de comando, como mostra abaixo:

```bash
jx create cluster gke
```

o detalhamento da criação está no tópico  do [Jenkins X](https://github.com/hebersonaguiar/ditodesafiodocs#jenkins-x)

## Jenkins X
O Jenkins X possui os conceitos de Aplicativos e Ambientes. Você não instala o Jenkins diretamente para usar o Jenkins X, pois o Jenkins é incorporado como um mecanismo de pipeline como parte da instalação.

Nesse projeto a toda a criação do cluster e as integrações do Kubernetes com Jenkins X, Chart Museum e Nexus, foi toda realizada utilizando o Jenkins X, porém antes da criação do cluster, é necessário realizar as instalações e configurações dos pré-requisitos, que são os jx, helm, kubectl e gcloud, segue abaixo:

* Jenkins X
```bash
JX_VERSION=v2.0.845
OS_ARCH=linux-amd64
curl -L https://github.com/jenkins-x/jx/releases/download/"${JX_VERSION}"/jx-"${OS_ARCH}".tar.gz | tar xzv
sudo mv jx /usr/local/bin
```

* gcloud
```bash
curl https://sdk.cloud.google.com | bash
exec -l $SHELL
```

* Helm e Kubectl

Para instalar o helm e kubectl, pode-se usar o próprio jx:
```bash
jx install dependencies
```
Ao executar o comando acima, o jx irá solicitar que seja escolhido as dependências que deseja instalar, basta escolher apertando a barra de espaço e depois enter par iniciar a instalação, como mostra a imagem abaixo:

![jx install dependencies kubectl helm](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/install-dependencies.png)
![jx install dependencies kubectl helm](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/install-dependencies-s.png)


Realizado a instlação dos pré-requisitos, chegou a hora de criar o cluster, no momento da instalação, são requisitdas diversas informações relevantes para criação do cluster como, autenticação no GCP com uma conta válida, zona de instalação do cluster, api key do github, tipo de instalação do Jenkins X, etc.

Para iniciar basta executar o comando abaixo:

```bash
jx create cluster gke
```

Abaixo segue o passo-a-passo utilizado para criação e configuração do cluster:

* Autenticação no GCP, o Jenkins X gera um link de acesso para autetnticação, basta copiar e colar em qualquer browser:
![autenticação gcp](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/solicitacao-de-acesso.png)

* Seleção do projeto para criação do cluster
![seleção do projeto gcp](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/projeto-escolha.png)

* Seleção da zona para criação do cluster
![selecao da zona gcp](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/zona.png)

* Criação do Cluster
![criação do cluster gcp](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/criando-cluster.png)

* Cluster Criado
![cluster criado gcp](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/cluster-criado.png)

* Seleção do tipo de instalação do Jenkins X
![tipo jenkins](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/tipo-jenkins.png)

* Domínio

	![dominio](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/dominio-jx.png)

* Usuário do GitHub

	![usuario do github](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/gitusername-jx.png)

* Criação de token de acesso do Github no Jenkins x
![token github jenkinsx](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/api-token-jx.png)

* Criação de token para uso do pipeline
![token github pipeline](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/api-token-jx-cicd.png)

* Criação de repositórios para deploy das aplicações
![criacao repos](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/criando-repos.png)
![criacao repos 2](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/criando-repos-git.png)

* Jenkins, Chart Museum, Nexus, Docker Registry criados com sucesso
![instalacao concluida](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/jenkins-instalado.png)

* JAcesso às aplicações criadas (Ingress)
![instalacao concluida](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/ingress-criado.png)

O acesso as aplicações criadas são:

`Username: admin`

`Password: i-HJomkfDA7~byKH429S`

Acessando as aplicações:

* Jenkins X - [jenkins.jx.108.59.87.39.nip.io](http://jenkins.jx.108.59.87.39.nip.io)
![jenkinsx](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/jenkins-access.png)

* Chart Museum - [chartmuseum.jx.108.59.87.39.nip.io](http://chartmuseum.jx.108.59.87.39.nip.io)
![chat museum](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/chatmuseum-access.png)

* Nexus - [nexus.jx.108.59.87.39.nip.io](http://nexus.jx.108.59.87.39.nip.io)
![nexus](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/nexus-access.png)


## Importação dos repositórios para o Jenkins X
Antes de importar os projetos para o Jenkins X, é necessário enteder a arquitetura da aplicação, a aplicação [Frontend](https://github.com/hebersonaguiar/ditochatfrontend) conecta na aplicação [Backend](https://github.com/hebersonaguiar/ditochatbackend) pra envio de mensagens, e o [Backend](https://github.com/hebersonaguiar/ditochatbackend) se conecta a um Redis para salvar as mensagens, como mostra a imagem abaixo:

![arquitetura](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/arquitetura-app.png)

O redis é uma aplicação de terceiro, com isso não foi criado um repositório para ele, dessa forma iremos criar um redis no cluster utilizando o kubernetes, segue abaixo:

* Criando namespace para o projeto
```bash
kubectl create namespace chatdito
```

* Criando o deployment do Redis 
```bash
kubectl run redis --port 6379 --image=redis:3.2.12-alpine -n chatdito
```

* Criando o serviço do Redis 
```bash
kubectl expose --name redis deployment redis --target-port=6379 --type=LoadBalancer -n chatdito
```

Pronto, o redis está funcionando:

![redis](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/redis-k8s.png)

Como mostra na imagem acima, o seviço do redis possui um ip e porta pública, agora iremos criar o aponamento dns [redis.ditochallenge.com](http://redis.ditochallenge.com) utilizando o Google Cloud DNS.
![dns redis](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/redis-dns.png)

Outra configuração importante a ser feita é a ciação do configmap da aplicação, no qual será informado as variáveis de conexão com o redis e o frontend:

```bash
kubectl create configmap chat-backend-values --from-literal ALLOWED_ORIGIN='http://frontend.ditochallenge.com:3000' --from-literal REDIS_ADDR='redis.ditochallenge.com:6379' -n chatdito
```

Com o redis funcionando e o configmap criado podemos então importar o repositório do [Backend](https://github.com/hebersonaguiar/ditochatbackend), para isso vamos executar o comando abaixo:

```bash
jx import --no-draft --url https://github.com/hebersonaguiar/ditochatbackend.git
```

* a tag `--no-draft` faz com que não seja criado os artefatos como o chart e Jenkinsfile, o repositório já possui esses artefatos. 

Importação do repositório do [Backend](https://github.com/hebersonaguiar/ditochatbackend) realizada com suceso:
![importacao backend](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/import-backend.png)

O próximo repositório a ser importado é o [Frontend](https://github.com/hebersonaguiar/ditochatfrontend), para isso vamos executar o comando abaixo:

```bash
jx import --no-draft --url https://github.com/hebersonaguiar/ditochatfrontend.git
```

* a tag `--no-draft` faz com que não seja criado os artefatos como o chart e Jenkinsfile, o repositório já possui esses artefatos. 

Importação do repositório do [Frontend](https://github.com/hebersonaguiar/ditochatfrontend) realizada com suceso:
![importacao frontend](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/import-frontend.png)

Após a importação dos repositórios, é dado início ao CI - Continuous Integration, nesse mommento  é realizado o build e push da imagem e do chart, para suas respecitvas aplicações Docker Registry e Chart Museum, feito isso a imagem é promovida para o ambiente de deploy Kubernetes, nesse mommento o repositóio de CD - Continuous Delivery é acionado após uma solicitação de pull request, realizado o pull request o deploy é realizado com a image e chart criado anteriormente.


## Fluxo CI/CD

## Prometheus
O Prometheus é um kit de ferramentas de monitoramento e alerta de sistemas de código aberto criado originalmente no SoundCloud. Desde a sua criação em 2012, muitas empresas e organizações adotaram o Prometheus, e o projeto possui uma comunidade de desenvolvedores e usuários muito ativa. Agora é um projeto de código aberto independente e mantido independentemente de qualquer empresa. Para enfatizar isso e esclarecer a estrutura de governança do projeto.

Iremos utilizar nesse projeto o prometheus para coletar dados do cluster kubernetes bem como as aplicações, para sua instalação iremos utilizar o helm chart, por ser uma aplicação de terceiro e também por está mantida sua versão estável no repositório de charts no github, para sua instalação iremos utilizar o comando abaixo:

* Criação de um namespace para o monitoramento e log 

```bash
kubectl create namespace monitoring-log
```

* Instalação do prometheus

```bash
helm install --name prometheus \
	--set server.persistentVolume.enabled=false,alertmanager.persistentVolume.enabled=false \
	--namespace monitoring-log \
	stable/prometheus
```

Após todos os serviços estarem funcionando, será necessário criar um ingress para o acesso externo, para isso iremos utilizar o comando abaixo:

```bash
kubectl create -f ingress-prometheus.yaml
```
O arquivo de configuação do ingress encontra-se em `conf/k8s/`

Prometheus em execução:

![prometheus](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/prometheus.png)


## Grafana
Grafana é uma suíte de análise e visualização métrica de código aberto. É mais comumente usado para visualizar dados de séries temporais para análise de infraestrutura e aplicativos.

Nesse projeto iremos istalar o grafana e configurá-lo para conectar-se ao prometheus e configurar dashboards de métricas do cluster e as aplicações, para isso iremos utilizar o helm chart,  para sua instalação iremos utilizar o comando abaixo:

* Criação de um namespace para o monitoramento e log (caso não exista ou deseja criar em outro)

```bash
kubectl create namespace monitoring-log
```

* Instalação do grafana

```bash
helm install --name grafana \
	--set persistence.enabled=false \
	--namespace monitoring-log \
	 stable/grafana
```

Após todos os serviços estarem funcionando, será necessário criar um ingress para o acesso externo, para isso iremos utilizar o comando abaixo:

```bash
kubectl create -f ingress-grafana.yaml
```
O arquivo de configuação do ingress encontra-se em `conf/k8s/`

Grafana em execução:

![grafana](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/grafana.png)

Na instalação do grafana é criado uma senha de acesso, para consulta-la execute o comando abaixo:

```bash
kubectl get secret --namespace monitoring-log grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Os acessos ao grafana são:

`Usuário: admin`

`Senha: gc5iMZLszrPVwLaJyEGxEf7nNvxL7uq0YGkLQU4p`

Criação de datasource de conexão com prometheus:
A criação da conexão do grafana com o prometheus é em Datasource > Prometheus, ao clicar vai abrir um formulário para preenchimento, segue abaixo as inforações para preenchimento:

`Nome: Prometheus`

`URL de conexão com prometheus: http://prometheus.ditochallenge.com`

Pronto, é só clicar em testar e salvar.
 

* Importação de dashboards
Dentro da pasta `conf/grafana/` possui alguns arquivos do tipo `.json` que são os dashboards que iremos utilizar, no qual mostra todas as infomrações do cluter, como, uso de memória, CPU, disco, etc, são métricas do cluster e dos contêiners. A importação de um dashboard é bem simples, no canto superior esquerdo, possui um ícone com o nome home, ao clicar será aberto um modal com algumas informações, entre elas a `Import Dashboard`, ao clicar uma nova página é aberta e algumas informações são solicitadas, que são como que o dashboard vai ser importado, possui três formas, Upload de um arquivo `.json`, o ID de um dashboard público ou colar o conteúdo de um arquivo `.json`, nesse caso, como temos um dashboard customizado iremos colar o contéudo do nosso arquivo `.json`, feito isso basta apenas clicar em load.

Agora para visualizar, basta ir em Home, clicar no dashboard "Kubernetes Cluster - Monitoramento" e visualizar os dados, como mostra a imagem abaixo:

![grafana](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/grafana-monitoring.png)