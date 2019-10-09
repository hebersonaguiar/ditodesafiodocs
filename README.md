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
