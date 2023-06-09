# Desafio BB

## Descrição
Construir um ambiente de streaming de dados utilizando Apache Nifi, Apache Kafka, Docker e Grafana.

A execução de todos esses componentes deve ser realizada com containers Docker.

1 - Você deve construir um fluxo no Nifi que realize os seguintes passos:

1.1 - Conectar e coletar os dados de uma fonte em formato streaming, exemplo:
- Twitter;
- Portais de notícias;
- Youtube. 
  
1.2 - Publicar estes dados coletados como eventos no Kafka. 
  
2 - Você deve construir um dashboard no Grafana para monitorar esse ambiente. O que será monitorado ficará ao seu critério.
Qualquer dúvida, estamos à disposição! 


## Solução implementada

### Descrição da solução

Construído um ambiente de streaming de dados utilizando Docker, Docker-Compose, Apache Nifi, Apache Kafka, ElasticSearch, Alert Manager, Prometeus e Grafana.

O fluxo implementado no nifi recupera a cada 1 minuto dados da temperatura atual nas capitais do Brasil.

Os dados são coletados a partir do serviço http://servicos.cptec.inpe.br/XML/capitais/condicoesAtuais.xml.

São consideradas as estações meteorológicas dos aeroportos das capitais.


### Pré-requisitos básicos para execução
- Linux based Debian Distro
- Docker
- Docker-Compose
- Git
- Terminal por linha de comando
- IDE
- curl
- Offset Explorer (Kafka Tool)
- Navegador Internet (Chrome ou Firefox)
- Acesso Internet

Todas as ferramentas devem estar devidamente operacionais.

Vesões utilizadas no desenvolvimento:
- Windows 10 Pro 22H2 19045.2604
- WSL Ubuntu-20.04
- Docker version 20.10.22, build 3a2c30b
- docker-compose version 1.24.1, build 4667896b
- git version 2.25.1
- Visual Studio Code 1.77.3
- Google Chrome 112.0.5615.137

### Acesso ao código fonte
O código fonte está localizado no GitHub em https://github.com/Arleandro/bb-big-data-desafio.

Para acessar o código é necessário executar o seguinte comando:
```shell
$ git config --global http.sslVerify false
$ git clone https://github.com/Arleandro/bb-big-data-desafio.git
```

> Utilizar o diretório raíz da aplicação como workdir nas instruções de execução da solução.

### Inicialização do ecosistema
1) Executar com docker-compose o arquivo docker-compose.yaml com o seguinte comando:
```shell
$ docker-compose up
```

### Parar execução do ecosistema
```shell
$ docker-compose down
```


### Serviços Container

| Container | Imagem Docker |
| --------- | ------------- |
| [Elasticsearch](https://github.com/elastic/elasticsearch) | docker.elastic.co/elasticsearch/elasticsearch:7.14.0 |
| [Apache Nifi](https://nifi.apache.org/) | apache/nifi:latest |
| [Apache ZooKeeper](https://zookeeper.apache.org/) | confluentinc/cp-zookeeper:latest |
| [Apache Kafka](https://kafka.apache.org/) | confluentinc/cp-kafka:latest |
| [Container Advisor](https://github.com/google/cadvisor)  | gcr.io/cadvisor/cadvisor |
| [Alertmanager](https://github.com/prometheus/alertmanager) | prom/alertmanager  |
| [Node Exporter](https://github.com/prometheus/node_exporter) | quay.io/prometheus/node-exporter:latest |
| [Prometheus](https://prometheus.io/) | prom/prometheus:v2.36.2 |
| [Grafana](https://grafana.com/) | grafana/grafana |


### Consoles Web Containers

| Container | URL | Usuário | Senha |
| ----------- | ----------- | ---------- | ---------- |
| [Apache Nifi](https://nifi.apache.org/) | https://localhost:8443 | admin | nifibigdatabb |
| [Container Advisor](https://github.com/google/cadvisor) | http://localhost:8080 |  |  |
| [Prometheus](https://prometheus.io/) | http://localhost:9090 | | |
| [Grafana](https://grafana.com/) | http://localhost:3000 | admin | grafana  |

### Outro acessos
#### Grafana
- [Graficos de monitoramento](http://localhost:3000/d/64nrElFmk/docker-prometheus-monitoring)


#### Prometheus
- [Targets](http://localhost:9090/targets)
- [Serviços monitorados](http://localhost:9090/graph?g0.expr=%0Aup&g0.tab=1&g0.stacked=0&g0.show_exemplars=0&g0.range_input=1h)
- [Alertas gerados](http://localhost:9090/alerts)


#### Container Advisor
- [Metricas geradas](http://localhost:8080/metrics)

### Apache Nifi - Fluxo
Projeto do fluxo disponível no arquivo [inpe-monitor-temperatura-capitais.xml](/resources/nifi/inpe-monitor-temperatura-capitais.xml)

O fluxo implementado no nifi recupera a cada 1 minuto dados da temperatura atual nas capitais do Brasil.

Os dados meteorológicos são obtidos a partir do serviço [Previsão de Tempo em XML](http://servicos.cptec.inpe.br/XML/capitais/condicoesAtuais.xml) do [CPTEC/INPE](http://servicos.cptec.inpe.br/XML/).

São consideradas as estações meteorológicas dos aeroportos das capitais.

**Para execução do fluxo é necessário:**
- Importar o projeto como template
- Habilitar Record Reader e Record Write do ConvertRecord no Process Group DadosEntrada.
- Habilitar Record Reader e Record Write do ConsumeKafkaRecord_1_0 no Process Group DadosSaida.
- Iniciar a execução do fluxo

> Não esquecer de parar a execução do processo após verificações.


### Elasticsearch - Informações

#### Datasource Grafana
Para o teste do datasouce do Elasticsearch funcionar no Grafana é necessário que exista o índice inpe_temperatura_capitais no grafana `com registros já realizados pelo fluxo do Nifi`.


#### Criar índice

O índice pode ser criado executando o seguinte comando: 
```shell
$ curl --location --request PUT 'http://localhost:9200/inpe_temperatura_capitais'
```

Response payload
```json
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "inpe_temperatura_capitais"
}
```

#### Excluir índice
Nas verificações e/ou testes da solução poderá, eventualmente, ser necessário a exclusão dos dados no Elasticsearch. Isso pode ser realizado apagando o índice já existente.

Para apagar o índice no elasticsearch, executar o seguinte comando:
```shell
$ curl --location --request DELETE 'http://localhost:9200/inpe_temperatura_capitais'
```

Response payload
```json
{
    "acknowledged": true
}
```


#### Consultar registros do índice
Para consultar os dados do índice utilizado no processamento, executar o seguinte comando:
```shell
$ curl --location 'http://localhost:9200/inpe_temperatura_capitais/_search?size=1000&pretty=true&q=*%3A*'
```

Response payload (Estruta de dados equivalente)
```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 26,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "inpe_temperatura_capitais",
                "_type": "json",
                "_id": "sHu5nIcBatnJ3VKRjdaf",
                "_score": 1.0,
                "_source": {
                    "codigo": "SBAR",
                    "atualizacao": "15/02/2023 22:00:00",
                    "pressao": 1010,
                    "temperatura": 28,
                    "tempo": "ps",
                    "tempo_desc": "PredomÃ­nio de Sol",
                    "umidade": 76,
                    "vento_dir": 80,
                    "vento_int": 18,
                    "intensidade": ">10000",
                    "timestamp": "2023-02-15T22:00:00"
                }
            }
        ]
    }
}
```
