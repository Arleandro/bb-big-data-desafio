# bb-big-data-desafio
Construir um ambiente de streaming de dados utilizando Apache Nifi, Apache Kafka, Docker e Grafana.

Pré-requisitos básicos:
- Docker
- Docker-Compose
- Linux Ubuntu
- Navegador Internet (Chrome ou Firefox)


Para Executar:

Adicionar permissão para o arquivo run.sh
$ sudo chmod +x run.sh

Executar o arquivo
$ ./run.sh
ou
$ docker-compose up

Para parar Execução
$ docker-compose down


Acessos às aplicações implementadas:

Grafana
Acesso à aplicação
http://localhost:3000

Graficos de monitoramento
http://localhost:3000/d/64nrElFmk/docker-prometheus-monitoring

Para o teste do datasouce do Elasticsearch funcionar no Grafana é necessário que exista o índice inpe-temperatura-capitais no grafana com registros já realizados.

O índice pode ser criado executando o seguinte comando: 
$ curl --location --request PUT 'http://localhost:9200/inpe-temperatura-capitais'
Response payload
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "inpe-temperatura-capitais"
}


Nifi
https://localhost:8443/nifi
Login
Usuário: admin
Senha: nifibigdatabb

Projeto do fluxo disponível em: ./resources/nifi/inpe-monitor-temperatura-capitais.xml

O fluxo implementado no nifi recupera a cada 1 minuto dados da temperatura atual nas capitais do Brasil.
Os dados são coletados a partir do serviço http://servicos.cptec.inpe.br/XML/capitais/condicoesAtuais.xml.
São consideradas as estações meteorológicas dos aeroportos das capitais.

Para execução do fluxo é necessário:
- Importar o projeto como template
- Habilitar Record Reader e Record Write do ConvertRecord no Process Group DadosEntrada.
- Habilitar Record Reader e Record Write do ConsumeKafkaRecord_1_0 no Process Group DadosSaida.
- Iniciar a execução do fluxo

Observação: não esquecer de parar a execução do processo após verificações.


Prometheus

Acesso à aplicação
http://localhost:9090/targets

Serviços monitorados
http://localhost:9090/graph?g0.expr=%0Aup&g0.tab=1&g0.stacked=0&g0.show_exemplars=0&g0.range_input=1h

Alertas gerados
http://localhost:9090/alerts



cadvisor
Acesso à aplicação
http://localhost:8080

Metricas geradas
http://localhost:8080/metrics


Elasticsearch
Para consultar os dados do índice utilizado no processamento, executar o seguinte comando:
$ curl --location 'http://localhost:9200/inpe-temperatura-capitais/_search?size=1000&pretty=true&q=*%3A*'
HTTP Status Code: 200
Response payload (Estruta de dados equivalente)
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
                "_index": "inpe-temperatura-capitais",
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


Para criar o índice no elasticsearch, executar o seguinte comando:
$ curl --location --request PUT 'http://localhost:9200/inpe-temperatura-capitais'
HTTP Status Code: 200
Response payload
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "inpe-temperatura-capitais"
}


Para apagar o índice no elasticsearch, executar o seguinte comando:
$ curl --location --request DELETE 'http://localhost:9200/inpe-temperatura-capitais'
HTTP Status Code: 200
Response payload
{
    "acknowledged": true
}
