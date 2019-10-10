Esse reposítório foi criado para servir de complementação à documetação dos projetos do Dito Desafio, que são os repositórios abaixo:

[Dito Chat Backend](https://github.com/hebersonaguiar/ditochatbackend)

[Dito Chat Frontend](https://github.com/hebersonaguiar/ditochatfrontend)

Os repositórios infomrados acima, possuem suas prórprias documentações referente ao seu uso local e em um ambiente de cluster kubernetes.

## A proposta
A proposta desse projeto é de em base ao repositório [Dito Chat](https://github.com/ditointernet/dito-chat) que foi repassado pela Dito, colocar em produção, prezando pela utilização de ferramentas de deploy contínuo, grantindo a alta disponibilidade e manutenabilidade(monitoramento e logs), para isso diversos trabalhos foram realizado, pois o repositório [Dito Chat](https://github.com/ditointernet/dito-chat) possuem apenas o código fonte da aplicação e uma breve explicação de como deve ser utilizada.

Para esse desafio foram utilizados as seguintes tecnologias:

[Docker](https://github.com/hebersonaguiar/ditodesafiodocs#docker)

[Google Cloud Plataform](https://github.com/hebersonaguiar/ditodesafiodocs#google-cloud-plataform)

[Kubernetes Engine](https://github.com/hebersonaguiar/ditodesafiodocs#kubernetes)

[Jenkins X](https://github.com/hebersonaguiar/ditodesafiodocs#jenkins-x)

[Helm Chart](https://github.com/hebersonaguiar/ditodesafiodocs#helm-chart)

[Chart Museum](https://github.com/hebersonaguiar/ditodesafiodocs#chart-museum)

[GitHub](https://github.com/hebersonaguiar/ditodesafiodocs#github)

[Skaffold](https://github.com/hebersonaguiar/ditodesafiodocs#skaffold)

[Docker Registry](https://github.com/hebersonaguiar/ditodesafiodocs#docker-registry)

[Prometheus](https://github.com/hebersonaguiar/ditodesafiodocs#prometheus)

[Grafana](https://github.com/hebersonaguiar/ditodesafiodocs#grafana)

[ELK](https://github.com/hebersonaguiar/ditodesafiodocs#elk)


Todos as tecnologias informadas acima serão descritas e detalhadas nesse documento, bem como suas integrações e uso.

## Qual ideia desse projeto?

De forma bem enxuta, a ideia desse projeto é que ao realizar alguma alteração no código da aplicação no repositório, automaticamente o deploy da aplicação para produção é acionada, mas como que isso funciona? Ao realizar o commit um webhook integrado com o Jenkins X é acionada dando inicio aó deploy, só que durante esse processo algumas ações são realizadas para que as alterações tenham efeito, que são o build da imagem da aplicação com as alterações realizadas, push da imagem para o repositório, build do helm chart com a nova versão da imagem, push do helm chart para o Chart Museum e por fim o deploy da aplicação para o cluster. 

Vale resaltar que, quando o commit é realizado, uma tag é criada e essa mesma tag segue todo o fluxo para todos os artefatos, imagem, helm chart, etc, ou seja, o rasterio da imagem dentro do fluxo é de fácil indentificação, apoiando muito na manutenção da aplicação e do ambiente.

Outro fato importante é, as informações de container como uso de memória, cpu e logs, são enviados para o Grafana e Kibana.

## Docker
O Docker é uma plataforma para desenvolvedores e administradores de sistemas para desenvolver, enviar e executar aplicativos. O Docker permite montar aplicativos rapidamente a partir de componentes e elimina o atrito que pode ocorrer no envio do código. O Docker permite que seu código seja testado e implantado na produção o mais rápido possível. Originalmente essa aplicação não foi desenvolvida para docker, porém sua criação é simples e rápido.

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

## Kubernetes

Kubernetes ou como é conhecido também K8s é um produto Open Source utilizado para automatizar a implantação, o dimensionamento e o gerenciamento de aplicativos em contêiner no qual agrupa contêineres que compõem uma aplicação em unidades lógicas para facilitar o gerenciamento e a descoberta de serviço

Nesse projeto foi utilizado o kubernetes na versão `v1.13.7-gke.24`, o cluster foi criado na GCP, o Kubernetes Engine não é voltado apenas para aplicativos sem estado. Com ele, é possível incluir armazenamento permanente e até mesmo executar um banco de dados no seu cluster.

A instação do cluster kubernetes foi realizada pelo Jenkins X, no qual possui uma forte integração com a GCP, criando um ambinete de CI/CD de forma rápida e fácil.

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

* Jenkins, Chart Museum, Docker Registry criados com sucesso
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

Nota: na aplicação é necessário informar alguns dados importantes para o funcionamento, que é a origem da conexão de envio das mensagens, ou seja, o [frontend](https://github.com/hebersonaguiar/ditochatfrontend) e o redis, que é o responsável por salvar as mensagens. Para esse projeto foi criado e configurado os domínios  `frontend.ditochallenge.com` e `redis.ditochallenge.com`, eles são informados no código da aplicação em `main.go`.

Com o redis funcionando podemos importar o repositório do [Backend](https://github.com/hebersonaguiar/ditochatbackend), para isso vamos executar o comando abaixo:

```bash
jx import --url https://github.com/hebersonaguiar/ditochatbackend.git
```

* realizado o import é necessário a alteração de alguns dados como, os values.yaml e o deployment.yaml de acordo com a aplicação.
* caso queira que não seja criado os artefatos use a tag `--no-draft` ela faz com que não seja criado os artefatos como o chart e Jenkinsfile. 


Importação do repositório do [Backend](https://github.com/hebersonaguiar/ditochatbackend) realizada com suceso:
![importacao backend](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/import-backend.png)

O próximo repositório a ser importado é o [Frontend](https://github.com/hebersonaguiar/ditochatfrontend), para isso vamos executar o comando abaixo:

```bash
jx import --url https://github.com/hebersonaguiar/ditochatfrontend.git
```

* realizado o import é necessário a alteração de alguns dados como, os values.yaml e o deployment.yaml de acordo com a aplicação.
* caso queira que não seja criado os artefatos use a tag `--no-draft` ela faz com que não seja criado os artefatos como o chart e Jenkinsfile. 

Importação do repositório do [Frontend](https://github.com/hebersonaguiar/ditochatfrontend) realizada com suceso:
![importacao frontend](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/import-frontend.png)

Após a importação dos repositórios, é dado início ao CI - Continuous Integration, nesse mommento  é realizado o build e push da imagem e do chart, para suas respecitvas aplicações Docker Registry e Chart Museum, feito isso a imagem é promovida para o ambiente de deploy Kubernetes, nesse mommento o repositóio de CD - Continuous Delivery é acionado após uma solicitação de pull request, realizado o pull request o deploy é realizado com a image e chart criado anteriormente.


## Fluxo CI/CD

## Helm Chart
O Helm é um gerenciador de aplicações Kubernetes onde cria, versiona, compartilha e pública os artefatos. Com ele é possível desenvolver templates dos arquivos YAML e durante a instalação de cada aplicação personalizar os parâmentros com facilidade. Nesse projeto o helm chart foi utilizado nos repositórios das aplicações [Frontend](https://github.com/hebersonaguiar/ditochatfrontend/tree/master/charts/ditochatfrontend) e [Backend](https://github.com/hebersonaguiar/ditochatbackend/tree/master/charts/ditochatbackend), no qual foi emcapsulado todos os arquivos necessários para a implantação das aplicações, como deployment, service, persistente volume, etc, um template padrão de uma aplicação é a seguinte:

![helm chart template](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/helm-chart-temp.png)

Dentro do diretório, existe um arquivo chamado `values.yaml`, muito importante dentro de um helm chart, ele é o resposável por informar para os arquivios YAML quais os valores que serão alterados que podem ser por exemplo: quantidade de replicas, portas de acesso, tipo de deploy, tamanho do volume a ser utilizado, etc.

Nesse projeto, o helm chart não foi criado pelo Jenkins X em sua importação, como informado nos tópicos anteriores, porém caso seja necessário basta remover a tag `--no-draft` no momento da instalação e o Jenkins X se encarrega de criar, entrentanto tenha cuidado, os valores padrões criados podem ser diferente do que a aplicação requer. 


## Chart Museum
O ChartMuseum é um servidor de repositório de Helm Chart de código aberto escrito em Go (Golang), com suporte para back-end de armazenamento em nuvem, incluindo Google Cloud Storage, Amazon S3, Microsoft Azure Blob Storage, Alibaba Cloud OSS Storage e Openstack Object Storage.

Nesse projeto o Chart Museum foi instalado no momento da instalção e configuração do Jenkins X e o Cluster. Ele é essencial para o funcionamento do fluxo CI/CD, onde todo encapsulamento da aplicação fica salva e versionada, podendo ser utilizada a qualquer momento. O Chart Museum usa uma api para sua manipulação para salvar, baixa e visualizar os chart, como mostra a imagem abaixo:

![importacao frontend](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/chart-museum.png)


## GitHub
GitHub é uma plataforma de hospedagem de código-fonte com controle de versão usando o Git. Ele permite que programadores, utilitários ou qualquer usuário cadastrado na plataforma contribuam em projetos privados e/ou Open Source de qualquer lugar do mundo.

Ele possui uma integação muito boa com o Jenkins X, apoiando e facilitando todo fluxo CI/CD bem como o desenvolvimento das aplicações e documentação.


## Skaffold
Skaffold é uma ferramenta de linha de comando que facilita o desenvolvimento contínuo de aplicações no Kubernetes. O Skaffold lida com o fluxo de trabalho para implantar a aplicação. No arquivo skaffold.yaml possuem as variáveis como a do registry, imagem, tag, helm chart, não é necessário nehnhuma alteração, elas são realizadas pelo Jenkins X.

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

[Grafana](http://grafana.ditochallenge.com) em execução:

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


## ELK
ELK signifca ELasticsearch, Logstash e Kíbana, um conjunto de aplicações que nos ajudam a ter uma melhor visialização dos logs de ambientes, em nosso projeto iremos configurar essas aplicações para que possamos ter os logs de nosso cluster.
Na instalação do ELK não iremos utilizar o Helm Chart, vamos fazer urilizando o próprio Kubernetes, todos as configurações que iremos utilizar aqui estão em `conf/k8s/elk`.

* Elasticssearch

O Elasticsearch é um mecanismo de pesquisa e análise de código aberto distribuído para todos os tipos de dados, incluindo texto, numérico, geoespacial, estruturado e não estruturado.
Primiero passo para instalação é a criação do deployment, no qual irá ser criado um pod do elasticsearch, para isso execute o comando abaixo:

```bash
kubectl create -f deployment-elastic.yaml
```

Após executar o comando, aguarde enquanto o pod é iniciado, após sua inicialização precisamos criar um serviço para o pod do elasticsearch, isso fará que o Logstash possa se comunicar com ele, para isso execute o comando abaixo:

```bash
kubectl create -f service-elastic.yaml
```

* Kibana

O Kibana permite visualizar os dados do Elasticsearch  para você poder fazer qualquer coisa, desde rastrear a carga de consultas até entender a maneira como as solicitações fluem pelos aplicativos.
O Kibana será nosso frontend dos logs, ou seja, iremos poder ver os logs, gráficos de logs de nosso cluster a partir dele, para sua instalação iremos executar os seguintes comandos:

```bash
kubectl create -f deployment-kibana.yaml
```

Após executar o comando, aguarde enquanto o pod é iniciado, após sua inicialização precisamos criar um serviço para o pod do kibana, execute o comando abaixo:

```bash
kubectl create -f service-kibana.yaml
```

Após a criação do serviço do kibana, iremos agora criar um ingress, o ingress irá nos permitir acessar o painel do kibana, execute o comando abaixo:

```bash
kubectl create -f ingress-kibana.yaml
```

* Logstash

O Logstash é um pipeline de processamento de dados open source do lado do servidor que faz a ingestão de dados a partir de inúmeras fontes simultaneamente, transforma-os e envia-os para o seu "esconderijo" favorito.
Para esse projeto iremos utilizar o Fluentd como Logstash, ele vai se conectar ao elasticsearch e coletar os dados, para isso iremos executar os comandos abaixo:

O comando abaixo cria umm RBAC (role-based access control), um controle de acesso para o Fluentd possa acessar corretamente todos os componentes do cluster.

```bash
kubectl create -f fluentd-rbac.yaml
```

Criado as permissões, agora podemos criar um DaemonSet, diferente do deployment, o DaemonSet fará com que todos os nós obrigatoriamente contenha um pod do Fluentd, isso é importante pois todos os nós precisam de um mecanismo que possa coletar os dados, segue abaixo o comando:

```bash
kubectl create -f fluentd-daemonset.yaml
```
Pronto, agora basta acessar o [Kibana](http://kibana.ditochallenge.com), configurar os index do logstash e pronto, é só criar os devidos gráficos e pesquisas. 

![elk](https://github.com/hebersonaguiar/ditodesafiodocs/blob/master/images/kibana-access.png)