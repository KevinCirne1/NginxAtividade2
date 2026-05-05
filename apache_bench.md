# Tutorial 2: Teste de Carga com Apache Bench (ab) via Docker

Este tutorial demonstra como configurar e executar testes de carga utilizando o **Apache Bench (ab)** em uma aplicação React provida pelo Nginx dentro de um ambiente Dockerizado.

## 🎯 Objetivo
Avaliar o desempenho do servidor Nginx simulando múltiplos usuários simultâneos acessando a Home da aplicação React.

## 🛠️ Passo a Passo com Docker

### Passo 1: Configuração do serviço ab_tester
No seu arquivo `docker-compose.yml`, adicione o serviço de testes. Utilizaremos uma imagem Alpine e instalaremos o `ab` ao iniciar o container:

```yaml
services:
  web:
    build: .
    container_name: react_nginx
    networks:
      - rede_teste

  ab_tester:
    image: alpine
    container_name: ab_tester
    # Instala o apache2-utils (que contém o ab) e mantém o container vivo
    command: sh -c "apk add --no-cache apache2-utils && sleep infinity"
    volumes:
      - .:/apps
    networks:
      - rede_teste
Passo 2: Entendendo a Comunicação
O container ab_tester se comunica com o container web através da rede interna do Docker.

O alvo do teste será: http://web/

Passo 3: Executando o Teste de Carga
Para disparar o teste com 1000 requisições totais e uma concorrência de 10 usuários, execute o comando abaixo no seu terminal (com os containers já rodando):

Bash
docker exec -it ab_tester ab -n 1000 -c 10 http://web/

Passo 4: Interpretando as Métricas Principais
Após o término do teste, o painel do Apache Bench exibirá vários resultados. 

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

Foque nestes dados principais:

Requests per second: Quantidade de requisições que o Nginx processou por segundo (Quanto maior, melhor).

Time per request: Tempo médio de resposta para cada usuário (Quanto menor, melhor).

Failed requests: Deve ser 0. Se houver falhas, o servidor está sobrecarregado ou a aplicação apresentou erro.

99% percentile: Indica que 99% das requisições foram atendidas abaixo de "X" milissegundos.