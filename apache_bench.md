# Tutorial 2: Teste de Carga com Apache Bench (ab) via Docker

Este tutorial detalha como configurar e utilizar a ferramenta **Apache Bench (ab)** para realizar testes de estresse e medir o desempenho da nossa aplicação React servida pelo Nginx, orquestrando tudo através do Docker Compose.

## 🎯 Objetivo
Nosso objetivo é simular um cenário de tráfego intenso na página Home (raiz) da aplicação. Para isso, vamos disparar **1000 requisições totais**, mantendo uma concorrência de **10 usuários simultâneos**, e observar como o Nginx lida com essa carga.

## 🛠️ Passo a Passo com Docker

Este guia assume que você já possui a estrutura base do projeto e que os serviços do `docker-compose.yml` estão em execução (comando `docker-compose up -d`).

### Passo 1: Configuração do Serviço `ab_tester`

Para manter a organização, não instalaremos o Apache Bench no container da aplicação. Em vez disso, criamos um serviço isolado no `docker-compose.yml` dedicado apenas aos testes:

```yaml
services:
  # ... (configuração do serviço 'web' omitida) ...

  ab_tester:
    build:
      context: ./ab
    container_name: ab_tester
    depends_on:
      - web
    networks:
      - app-network
O que cada linha faz:

build: context: ./ab: Instrui o Docker a procurar um Dockerfile dentro da pasta ./ab. Este arquivo deve conter as instruções para instalar o pacote apache2-utils.

depends_on: [web]: Garante uma ordem de inicialização correta, impedindo que o teste rode antes do servidor Nginx estar no ar.

networks: [app-network]: Insere o container de teste na mesma rede virtual da aplicação, permitindo a comunicação entre eles.

Passo 2: Acessando o Alvo (Comunicação Interna)
A comunicação entre containers na mesma rede Docker não passa pela internet externa, mas sim por um DNS interno provido pelo próprio Docker.

❌ Por que não usar http://localhost:8080? Porque, dentro do contexto do container ab_tester, a palavra "localhost" apontaria para ele mesmo, e não para o servidor Nginx.

✅ A forma correta é usar http://web/: O Docker automaticamente resolve o nome do serviço (web) para o IP interno correto do container Nginx, acessando diretamente a porta 80.

Passo 3: Disparando o Teste de Carga
Para executar o comando do Apache Bench de forma isolada e descartável, utilizamos o docker-compose run:

Bash
docker-compose run ab_tester -n 1000 -c 10 http://web/
Entendendo os parâmetros:

docker-compose run ab_tester: Levanta o container de teste especificamente para rodar a instrução a seguir.

-n 1000: Determina o número total de requisições do teste.

-c 10: Define a concorrência (10 requisições sendo disparadas ao mesmo tempo, repetidamente, até atingir as 1000).

http://web/: A URL alvo do nosso teste.

Passo 4: Analisando os Resultados
Após a conclusão, o Apache Bench gerará um relatório detalhado no seu terminal. Abaixo está um exemplo de resultado:

Plaintext
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       0    1   0.3      1       3
Processing:    1    2   1.2      2      12
Waiting:       0    1   1.1      1      10
Total:         1    3   1.3      3      14

Percentage of the requests served within a certain time (ms)
  50%      3
  66%      4
  75%      4
  80%      5
  90%      6
  95%      7
  98%      8
  99%     10
 100%     14 (longest request)

...
Requests per second:    312.45 [#/sec] (mean)
Time per request:       32.005 [ms] (mean)
Time per request:       3.200 [ms] (mean, across all concurrent requests)
Transfer rate:          1615.12 [Kbytes/sec] received

Failed requests:        0
Métricas cruciais para a análise:

Requests per second (RPS): No nosso exemplo, 312.45. Esta é a "vazão" do servidor. Indica que o Nginx conseguiu processar e devolver mais de 300 requisições a cada segundo.

Failed requests: 0. O cenário ideal. Indica que o servidor não gargalou a ponto de recusar conexões ou retornar erros 5xx.

Time per request (média global): 32.005 ms. O tempo médio de espera do ponto de vista do grupo de usuários simultâneos.

99% percentile: 10 ms. Demonstra uma ótima estabilidade, significando que 99% de todas as requisições foram resolvidas em, no máximo, 10 milissegundos.
