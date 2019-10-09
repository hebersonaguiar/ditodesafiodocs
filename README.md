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

Nesse projeto a toda a criação do cluster e as integrações do Kubernetes com Jenkins X, Chart Museum e Nexus, foi toda realizada utilizando o comando abaixo:

```bash
jx create cluster gke
```

porém, antes da criação do cluster, é necessário realizar as instalações e configurações do pré-requisitos, que são os jx, helm, kubectl e gcloud, segue abaixo:

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

![jx install dependencies kubectl helm]()
![jx install dependencies kubectl helm]()


No momento da instalação, são requisitdas diversas informações para criar o cluster e