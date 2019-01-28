---
layout: default
title: "Comparação de performance entre Node.js, Node.js Express, Vert.x e Spring WebFlux"
date: 2019-01-24
categories: [posts]
tags: [performance, vert.x, spring webflux, node.js, express, wrk, httperf]
---

Enquanto eu estudava sobre o desenvolvimento de APIs Rest com Node.Js, encontrei alguns artigos que indicam uma menor performance das APIs que utilizam o framework **Express**, quando comparadas às aplicações NodeJs puro. E no ímpeto de observar isso com meus próprios olhos, realizei alguns testes  simples com as ferramentas wrk e httperf, no meu próprio notebook. Adicionalmente, para deixar essa  brincadeira ainda mais divertida, incluí testes com Vert.x e Spring Webflux, os quais são frameworks da plataforma Java, que também se propõem ao processamento de requisições HTTP com **Event-loop**.

Para execução dos testes, criei quatro pequenas aplicações, que recebem uma requisição http GET, extraem da requisição o parâmetro "userName" e respondem com uma frase concatenando o userName recebido. Para melhorar a confiabilidade nos resultados, eu garanti que todas as quatro aplicações respondam com uma mensagem exatamente do mesmo tamanho (148 bytes). 
As aplicações usadas no teste estão disponíveis no GitHub <https://github.com/arrodrigues/benchmarking-experiments-on-simple-rest> .

Os seguintes passos foram executados em cada uma das quatro aplicações:
1. Iniciar a aplicação
2. Enviar requisições durante 1 minuto usando 100 conexões pelo `wrk`, com o objetivo de fazer o *warm up* da aplicação e desprezar os resultados.
3. Iniciar o monitoramento do uso de CPU pelo processo da aplicação, com a ferramenta `pidstat`.
4. Enviar requisições com o `wrk` novamente, durante 1 minuto, usando 100 conexões e coletar os resultados.
5. Com a ferramenta `httperf`, enviar 20.000 requisições, em 100 conexões, totalizando 2.000.000 e coletar os resultados.
6. Parar a aplicação.

As ferramentas `wrk` e `httperf` servem para gerar grandes cargas de requisições, e dado que elas apresentam os resultados de forma diferente e eu achei interessante comparar os resultados das duas. O `wrk` dispara o máximo de requisições possíveis durante um determinado tempo, no final ele indica quantas requisições foram respondidas. Já o `httperf`, envia uma quantidade determinada de requisições e apresenta quanto tempo foi gasto para completar todas elas.

Segue o roteiro completo, executado à partir do diretório raiz da aplicação: 
> Para a execução do roteiro, serão necessários 3 terminais Linux/Unix, e em cada um dos passos indica-se qual terminal deverá receber o comando. Além disso, para rodar as aplicações, você precisará do *npm* instalado na máquina e também do *maven*, com *java 8* ou maior. Cada uma das aplicações, quando iniciada, exibe no console o id do processo (PID) , e esse PID deverá ser usado para preencher o passo *3* do roteiro nas quatro aplicações.

{% capture roteiro-capture %}
```
1. APP_DIR=nodejs-express                                           (term 1, 2, 3)
2. npm start --prefix $APP_DIR                                      (term 1)
3. APP_PID=**ID DO PROCESSO**                                       (term 2, 3)
4. wrk -d 1m -c 100 -t 100 http://localhost:3000/?userName=Antonio  (term 2)
5. pidstat -u -p $APP_PID 1 > "${APP_DIR}_cpu_stats.txt"            (term 3) 
6. wrk -d 1m -c 100 -t 100 http://localhost:3000/?userName=Antonio > "wrk_${APP_DIR}_d1m_c100_t100.txt" (term 2)
7. httperf --server=localhost --port=3000 --uri=/?userName=Antonio --num-conns=100 --num-calls=20000 >  "httperf_${APP_DIR}_num-conns10_num-calls20000.txt" (term 2)
8. stop term 1 and 3


1. APP_DIR=nodejs-pure                                              (term 1, 2, 3)
2. npm start --prefix $APP_DIR                                      (term 1)
3. APP_PID=**ID DO PROCESSO**                                       (term 2, 3)
4. wrk -d 1m -c 100 -t 100 http://localhost:3000/?userName=Antonio  (term 2)
5. pidstat -u -p $APP_PID 1 > "${APP_DIR}_cpu_stats.txt"            (term 3) 
6. wrk -d 1m -c 100 -t 100 http://localhost:3000/?userName=Antonio > "wrk_${APP_DIR}_d1m_c100_t100.txt" (term 2)
7. httperf --server=localhost --port=3000 --uri=/?userName=Antonio --num-conns=100 --num-calls=20000 >  "httperf_${APP_DIR}_num-conns10_num-calls20000.txt" (term 2)
8. stop term 1 and 31


1. APP_DIR=java-spring-webflux                                      (term 1, 2, 3)
2. mvn -f $APP_DIR/pom.xml clean compile exec:exec                  (term 1)
3. APP_PID=**ID DO PROCESSO**                                       (term 2, 3)
4. wrk -d 1m -c 100 -t 100 http://localhost:3000/?userName=Antonio  (term 2)
5. pidstat -u -p $APP_PID 1 > "${APP_DIR}_cpu_stats.txt"            (term 3) 
6. wrk -d 1m -c 100 -t 100 http://localhost:3000/?userName=Antonio > "wrk_${APP_DIR}_d1m_c100_t100.txt" (term 2)
7. httperf --server=localhost --port=3000 --uri=/?userName=Antonio --num-conns=100 --num-calls=20000 >  "httperf_${APP_DIR}_num-conns10_num-calls20000.txt" (term 2)
8. stop term 1 and 3


1. APP_DIR=java-vertx                                               (term 1, 2, 3)
2. mvn -f $APP_DIR/pom.xml clean compile exec:exec                  (term 1)
3. APP_PID=**ID DO PROCESSO**                                       (term 2, 3)
4. wrk -d 1m -c 100 -t 100 http://localhost:3000/?userName=Antonio  (term 2)
5. pidstat -u -p $APP_PID 1 > "${APP_DIR}_cpu_stats.txt"            (term 3) 
6. wrk -d 1m -c 100 -t 100 http://localhost:3000/?userName=Antonio > "wrk_${APP_DIR}_d1m_c100_t100.txt" (term 2)
7. httperf --server=localhost --port=3000 --uri=/?userName=Antonio --num-conns=100 --num-calls=20000 >  "httperf_${APP_DIR}_num-conns10_num-calls20000.txt" (term 2)
8. stop term 1 and 3
```
{% endcapture %}
{% include widgets/toggle-panel.html toggle-name="roteiro-toggle" button-text="Clique para ver o roteiro" toggle-text=roteiro-capture  footer="Roteiro" %}

Ao fim desse roteiro, temos 3 arquivos de dados coletados para cada aplicação, totalizando 12 arquivos:



## Resultados do `wrk`
---
##### Aplicação NodeJS com Express (nodejs-express)
```
Running 1m test @ http://localhost:3000/?userName=Antonio
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.62ms  568.12us  21.84ms   94.31%
    Req/Sec   277.50     21.42     1.70k    83.44%
  1659466 requests in 1.00m, 234.22MB read
Requests/sec:  27637.66
Transfer/sec:      3.90MB

```
##### Aplicação somente NodeJS (nodejs-pure)
```
Running 1m test @ http://localhost:3000/?userName=Antonio
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.20ms  456.61us  15.90ms   95.69%
    Req/Sec   314.33     25.53     1.06k    85.82%
  1878280 requests in 1.00m, 265.11MB read
Requests/sec:  31283.18
Transfer/sec:      4.42MB
```
##### Aplicação Java com WebFlux (java-spring-webflux)
```
Running 1m test @ http://localhost:3000/?userName=Antonio
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.41ms    3.01ms  56.95ms   88.64%
    Req/Sec   341.72     67.97     3.36k    67.74%
  2043118 requests in 1.00m, 288.38MB read
Requests/sec:  34005.94
Transfer/sec:      4.80MB
```
##### Aplicação Java com Vert.X (java-vertx)
```
Running 1m test @ http://localhost:3000/?userName=Antonio
  100 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.47ms  471.22us  17.43ms   90.73%
    Req/Sec   694.93    101.64     5.55k    93.10%
  4152420 requests in 1.00m, 586.09MB read
Requests/sec:  69094.37
Transfer/sec:      9.75MB
```

## Resultados do `httperf`
---

##### Aplicação NodeJS com Express (nodejs-express)
```
httperf --client=0/1 --server=localhost --port=3000 --uri=/?userName=Antonio --send-buffer=4096 --recv-buffer=16384 --num-conns=100 --num-calls=20000
Maximum connect burst length: 1

Total: connections 100 requests 2000000 replies 2000000 test-duration 67.463 s

Connection rate: 1.5 conn/s (674.6 ms/conn, <=1 concurrent connections)
Connection time [ms]: min 664.9 avg 674.6 max 718.0 median 673.5 stddev 7.2
Connection time [ms]: connect 0.1
Connection length [replies/conn]: 20000.000

Request rate: 29646.0 req/s (0.0 ms/req)
Request size [B]: 79.0

Reply rate [replies/s]: min 29582.2 avg 29665.2 max 29815.2 stddev 77.2 (13 samples)
Reply time [ms]: response 0.0 transfer 0.0
Reply size [B]: header 104.0 content 44.0 footer 0.0 (total 148.0)
Reply status: 1xx=0 2xx=2000000 3xx=0 4xx=0 5xx=0

CPU time [s]: user 16.38 system 51.08 (user 24.3% system 75.7% total 100.0%)
Net I/O: 6571.9 KB/s (53.8*10^6 bps)

Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
Errors: fd-unavail 0 addrunavail 0 ftab-full 0 other 0
```
##### Aplicação Somente NodeJS (nodejs-pure)
```
httperf --client=0/1 --server=localhost --port=3000 --uri=/?userName=Antonio --send-buffer=4096 --recv-buffer=16384 --num-conns=100 --num-calls=20000
Maximum connect burst length: 1

Total: connections 100 requests 2000000 replies 2000000 test-duration 57.633 s

Connection rate: 1.7 conn/s (576.3 ms/conn, <=1 concurrent connections)
Connection time [ms]: min 550.2 avg 576.3 max 720.5 median 584.5 stddev 24.2
Connection time [ms]: connect 0.1
Connection length [replies/conn]: 20000.000

Request rate: 34702.3 req/s (0.0 ms/req)
Request size [B]: 79.0

Reply rate [replies/s]: min 33792.9 avg 34739.1 max 36132.4 stddev 990.0 (11 samples)
Reply time [ms]: response 0.0 transfer 0.0
Reply size [B]: header 104.0 content 44.0 footer 0.0 (total 148.0)
Reply status: 1xx=0 2xx=2000000 3xx=0 4xx=0 5xx=0

CPU time [s]: user 13.36 system 44.27 (user 23.2% system 76.8% total 100.0%)
Net I/O: 7692.8 KB/s (63.0*10^6 bps)

Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
Errors: fd-unavail 0 addrunavail 0 ftab-full 0 other 0

```
##### Aplicação Java com WebFlux (java-spring-webflux)
```
httperf --client=0/1 --server=localhost --port=3000 --uri=/?userName=Antonio --send-buffer=4096 --recv-buffer=16384 --num-conns=100 --num-calls=20000
Maximum connect burst length: 1

Total: connections 100 requests 2000000 replies 2000000 test-duration 109.594 s

Connection rate: 0.9 conn/s (1095.9 ms/conn, <=1 concurrent connections)
Connection time [ms]: min 1023.3 avg 1095.9 max 1301.9 median 1091.5 stddev 41.9
Connection time [ms]: connect 0.1
Connection length [replies/conn]: 20000.000

Request rate: 18249.1 req/s (0.1 ms/req)
Request size [B]: 79.0

Reply rate [replies/s]: min 17578.1 avg 18240.9 max 19334.2 stddev 441.2 (21 samples)
Reply time [ms]: response 0.0 transfer 0.0
Reply size [B]: header 104.0 content 44.0 footer 0.0 (total 148.0)
Reply status: 1xx=0 2xx=2000000 3xx=0 4xx=0 5xx=0

CPU time [s]: user 25.11 system 84.46 (user 22.9% system 77.1% total 100.0%)
Net I/O: 4045.5 KB/s (33.1*10^6 bps)

Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
Errors: fd-unavail 0 addrunavail 0 ftab-full 0 other 0

```

##### Aplicação Java com Vert.X (java-vertx)
```
httperf --client=0/1 --server=localhost --port=3000 --uri=/?userName=Antonio --send-buffer=4096 --recv-buffer=16384 --num-conns=100 --num-calls=20000
Maximum connect burst length: 1

Total: connections 100 requests 2000000 replies 2000000 test-duration 43.622 s

Connection rate: 2.3 conn/s (436.2 ms/conn, <=1 concurrent connections)
Connection time [ms]: min 416.2 avg 436.2 max 490.8 median 431.5 stddev 15.6
Connection time [ms]: connect 0.1
Connection length [replies/conn]: 20000.000

Request rate: 45848.2 req/s (0.0 ms/req)
Request size [B]: 79.0

Reply rate [replies/s]: min 44681.0 avg 45974.1 max 47239.3 stddev 811.0 (8 samples)
Reply time [ms]: response 0.0 transfer 0.0
Reply size [B]: header 104.0 content 44.0 footer 0.0 (total 148.0)
Reply status: 1xx=0 2xx=2000000 3xx=0 4xx=0 5xx=0

CPU time [s]: user 9.82 system 33.79 (user 22.5% system 77.5% total 100.0%)
Net I/O: 10163.6 KB/s (83.3*10^6 bps)

Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
Errors: fd-unavail 0 addrunavail 0 ftab-full 0 other 0
```

## Resultados do `pidstat`
---
{% capture nodejs-express-pidstats %}
```
04:22:27 PM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
04:22:28 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:29 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:30 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:31 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:32 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:33 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:34 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:35 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:36 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:37 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:38 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:39 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:40 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:41 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:42 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:43 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:44 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:45 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:46 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:47 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:48 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:49 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:50 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:51 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:52 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:53 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:54 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:55 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:56 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:57 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:58 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:22:59 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:00 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:01 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:02 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:03 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:04 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:05 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:06 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:07 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:08 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:09 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:10 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:11 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:12 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:13 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:14 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:15 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:16 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:17 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:18 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:19 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:20 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:21 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:22 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:23 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:24 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:25 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:26 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:27 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:28 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:29 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:30 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:31 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:32 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:33 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     1  node
04:23:34 PM  1000     12705   47,00   10,00    0,00    0,00   57,00     1  node
04:23:35 PM  1000     12705   76,00   31,00    0,00    0,00  100,00     0  node
04:23:36 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     1  node
04:23:37 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     1  node
04:23:38 PM  1000     12705   74,00   29,00    0,00    0,00  100,00     3  node
04:23:39 PM  1000     12705   73,00   30,00    0,00    0,00  100,00     3  node
04:23:40 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     1  node
04:23:41 PM  1000     12705   70,00   32,00    0,00    0,00  100,00     1  node
04:23:42 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     3  node
04:23:43 PM  1000     12705   72,00   31,00    0,00    0,00  100,00     2  node
04:23:44 PM  1000     12705   74,00   29,00    0,00    1,00  100,00     0  node
04:23:45 PM  1000     12705   66,00   36,00    0,00    0,00  100,00     2  node
04:23:46 PM  1000     12705   76,00   26,00    0,00    0,00  100,00     1  node
04:23:47 PM  1000     12705   73,00   29,00    0,00    0,00  100,00     3  node
04:23:48 PM  1000     12705   70,00   32,00    0,00    1,00  100,00     2  node
04:23:49 PM  1000     12705   71,00   32,00    0,00    0,00  100,00     3  node
04:23:50 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     1  node
04:23:51 PM  1000     12705   65,00   38,00    0,00    0,00  100,00     0  node
04:23:52 PM  1000     12705   74,00   28,00    0,00    0,00  100,00     0  node
04:23:53 PM  1000     12705   72,00   30,00    0,00    1,00  100,00     1  node
04:23:54 PM  1000     12705   76,00   26,00    0,00    0,00  100,00     0  node
04:23:55 PM  1000     12705   76,00   27,00    0,00    0,00  100,00     0  node
04:23:56 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     3  node
04:23:57 PM  1000     12705   70,00   32,00    0,00    0,00  100,00     1  node
04:23:58 PM  1000     12705   74,00   29,00    0,00    0,00  100,00     2  node
04:23:59 PM  1000     12705   75,00   28,00    0,00    0,00  100,00     1  node
04:24:00 PM  1000     12705   75,00   26,00    0,00    0,00  100,00     2  node
04:24:01 PM  1000     12705   70,00   33,00    0,00    0,00  100,00     0  node
04:24:02 PM  1000     12705   74,00   28,00    0,00    0,00  100,00     0  node
04:24:03 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     2  node
04:24:04 PM  1000     12705   74,00   29,00    0,00    0,00  100,00     2  node
04:24:05 PM  1000     12705   74,00   29,00    0,00    0,00  100,00     2  node
04:24:06 PM  1000     12705   77,00   25,00    0,00    1,00  100,00     0  node
04:24:07 PM  1000     12705   74,00   28,00    0,00    0,00  100,00     0  node
04:24:08 PM  1000     12705   72,00   31,00    0,00    0,00  100,00     2  node
04:24:09 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     1  node
04:24:10 PM  1000     12705   71,00   31,00    0,00    0,00  100,00     2  node
04:24:11 PM  1000     12705   75,00   28,00    0,00    0,00  100,00     3  node
04:24:12 PM  1000     12705   73,00   30,00    0,00    0,00  100,00     3  node
04:24:13 PM  1000     12705   76,00   26,00    0,00    0,00  100,00     0  node
04:24:14 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     2  node
04:24:15 PM  1000     12705   74,00   29,00    0,00    0,00  100,00     2  node
04:24:16 PM  1000     12705   81,00   21,00    0,00    0,00  100,00     3  node
04:24:17 PM  1000     12705   69,00   33,00    0,00    0,00  100,00     1  node
04:24:18 PM  1000     12705   72,00   31,00    0,00    0,00  100,00     2  node
04:24:19 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     2  node
04:24:20 PM  1000     12705   76,00   26,00    0,00    1,00  100,00     1  node
04:24:21 PM  1000     12705   74,00   28,00    0,00    0,00  100,00     1  node
04:24:22 PM  1000     12705   78,00   25,00    0,00    0,00  100,00     3  node
04:24:23 PM  1000     12705   69,00   34,00    0,00    0,00  100,00     3  node
04:24:24 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     2  node
04:24:25 PM  1000     12705   70,00   32,00    0,00    0,00  100,00     1  node
04:24:26 PM  1000     12705   70,00   32,00    0,00    0,00  100,00     1  node
04:24:27 PM  1000     12705   73,00   30,00    0,00    0,00  100,00     1  node
04:24:28 PM  1000     12705   76,00   26,00    0,00    0,00  100,00     3  node
04:24:29 PM  1000     12705   71,00   31,00    0,00    0,00  100,00     3  node
04:24:30 PM  1000     12705   75,00   28,00    0,00    1,00  100,00     1  node
04:24:31 PM  1000     12705   76,00   26,00    0,00    0,00  100,00     3  node
04:24:32 PM  1000     12705   72,00   31,00    0,00    0,00  100,00     0  node
04:24:33 PM  1000     12705   74,00   28,00    0,00    0,00  100,00     0  node
04:24:34 PM  1000     12705   47,00   21,00    0,00    0,00   68,00     2  node
04:24:35 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:36 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:37 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:38 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:39 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:40 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:41 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:42 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:43 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:44 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:45 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:46 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:47 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:48 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:49 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:50 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:51 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:52 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:53 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:54 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:55 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:56 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:57 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:58 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:24:59 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:25:00 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:25:01 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:25:02 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:25:03 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:25:04 PM  1000     12705    8,00    5,00    0,00    0,00   13,00     3  node
04:25:05 PM  1000     12705   66,00   36,00    0,00    0,00  100,00     1  node
04:25:06 PM  1000     12705   73,00   29,00    0,00    0,00  100,00     1  node
04:25:07 PM  1000     12705   71,00   31,00    0,00    0,00  100,00     1  node
04:25:08 PM  1000     12705   70,00   32,00    0,00    0,00  100,00     1  node
04:25:09 PM  1000     12705   67,00   35,00    0,00    0,00  100,00     1  node
04:25:10 PM  1000     12705   72,00   29,00    0,00    0,00  100,00     1  node
04:25:11 PM  1000     12705   73,00   28,00    0,00    0,00  100,00     1  node
04:25:12 PM  1000     12705   77,00   26,00    0,00    0,00  100,00     1  node
04:25:13 PM  1000     12705   68,00   33,00    0,00    0,00  100,00     3  node
04:25:14 PM  1000     12705   75,00   28,00    0,00    0,00  100,00     3  node
04:25:15 PM  1000     12705   76,00   25,00    0,00    0,00  100,00     3  node
04:25:16 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     3  node
04:25:17 PM  1000     12705   76,00   25,00    0,00    0,00  100,00     3  node
04:25:18 PM  1000     12705   71,00   32,00    0,00    0,00  100,00     3  node
04:25:19 PM  1000     12705   78,00   24,00    0,00    0,00  100,00     3  node
04:25:20 PM  1000     12705   75,00   26,00    0,00    0,00  100,00     3  node
04:25:21 PM  1000     12705   78,00   24,00    0,00    0,00  100,00     3  node
04:25:22 PM  1000     12705   77,00   24,00    0,00    0,00  100,00     3  node
04:25:23 PM  1000     12705   70,00   32,00    0,00    0,00  100,00     0  node
04:25:24 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     0  node
04:25:25 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     2  node
04:25:26 PM  1000     12705   75,00   26,00    0,00    0,00  100,00     0  node
04:25:27 PM  1000     12705   72,00   31,00    0,00    0,00  100,00     2  node
04:25:28 PM  1000     12705   72,00   29,00    0,00    0,00  100,00     0  node
04:25:29 PM  1000     12705   71,00   30,00    0,00    0,00  100,00     0  node
04:25:30 PM  1000     12705   67,00   35,00    0,00    0,00  100,00     0  node
04:25:31 PM  1000     12705   73,00   29,00    0,00    0,00  100,00     0  node
04:25:32 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     0  node
04:25:33 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     0  node
04:25:34 PM  1000     12705   78,00   24,00    0,00    0,00  100,00     1  node
04:25:35 PM  1000     12705   71,00   31,00    0,00    0,00  100,00     1  node
04:25:36 PM  1000     12705   74,00   27,00    0,00    0,00  100,00     2  node
04:25:37 PM  1000     12705   74,00   28,00    0,00    0,00  100,00     2  node
04:25:38 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     0  node
04:25:39 PM  1000     12705   67,00   35,00    0,00    0,00  100,00     0  node
04:25:40 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     0  node
04:25:41 PM  1000     12705   73,00   28,00    0,00    0,00  100,00     0  node
04:25:42 PM  1000     12705   70,00   32,00    0,00    0,00  100,00     0  node
04:25:43 PM  1000     12705   68,00   33,00    0,00    0,00  100,00     0  node
04:25:44 PM  1000     12705   73,00   29,00    0,00    0,00  100,00     2  node
04:25:45 PM  1000     12705   67,00   35,00    0,00    1,00  100,00     2  node
04:25:46 PM  1000     12705   71,00   31,00    0,00    0,00  100,00     0  node
04:25:47 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     1  node
04:25:48 PM  1000     12705   73,00   29,00    0,00    0,00  100,00     2  node
04:25:49 PM  1000     12705   72,00   29,00    0,00    0,00  100,00     2  node
04:25:50 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     2  node
04:25:51 PM  1000     12705   71,00   30,00    0,00    0,00  100,00     3  node
04:25:52 PM  1000     12705   73,00   29,00    0,00    0,00  100,00     2  node
04:25:53 PM  1000     12705   76,00   26,00    0,00    0,00  100,00     2  node
04:25:54 PM  1000     12705   73,27   27,72    0,00    0,00  100,00     2  node
04:25:55 PM  1000     12705   74,00   28,00    0,00    0,00  100,00     2  node
04:25:56 PM  1000     12705   71,00   30,00    0,00    0,00  100,00     0  node
04:25:57 PM  1000     12705   74,00   29,00    0,00    0,00  100,00     0  node
04:25:58 PM  1000     12705   61,00   40,00    0,00    0,00  100,00     0  node
04:25:59 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     0  node
04:26:00 PM  1000     12705   75,00   27,00    0,00    0,00  100,00     2  node
04:26:01 PM  1000     12705   74,00   27,00    0,00    0,00  100,00     2  node
04:26:02 PM  1000     12705   71,00   31,00    0,00    0,00  100,00     2  node
04:26:03 PM  1000     12705   70,00   32,00    0,00    0,00  100,00     2  node
04:26:04 PM  1000     12705   74,00   27,00    0,00    0,00  100,00     0  node
04:26:05 PM  1000     12705   74,00   28,00    0,00    0,00  100,00     0  node
04:26:06 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     2  node
04:26:07 PM  1000     12705   74,00   27,00    0,00    0,00  100,00     0  node
04:26:08 PM  1000     12705   67,00   35,00    0,00    0,00  100,00     0  node
04:26:09 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     0  node
04:26:10 PM  1000     12705   80,00   29,00    0,00    0,00  100,00     2  node
04:26:11 PM  1000     12705   72,00   30,00    0,00    0,00  100,00     2  node
04:26:12 PM  1000     12705   24,00   11,00    0,00    0,00   35,00     2  node
04:26:13 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:26:14 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:26:15 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:26:16 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:26:17 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:26:18 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node
04:26:19 PM  1000     12705    0,00    0,00    0,00    0,00    0,00     2  node

Average:     1000     12705   40,10   16,12    0,00    0,03   56,21     -  node

```
{% endcapture %}
{% include widgets/toggle-panel.html toggle-name="nodejs-express-pidstats-toggle" button-text="NodeJS com Express" toggle-text=nodejs-express-pidstats  footer="Fim do pidstats do NodeJS com Express" %}


{% capture nodejs-pure-pidstats %}
```
04:31:39 PM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
04:31:40 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:41 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:42 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:43 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:44 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:45 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:46 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:47 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:48 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:49 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:50 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:51 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:52 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     2  node
04:31:53 PM  1000     13703   65,00   23,00    0,00    0,00   88,00     0  node
04:31:54 PM  1000     13703   72,00   31,00    0,00    0,00  100,00     1  node
04:31:55 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     3  node
04:31:56 PM  1000     13703   72,00   29,00    0,00    0,00  100,00     0  node
04:31:57 PM  1000     13703   73,00   29,00    0,00    0,00  100,00     0  node
04:31:58 PM  1000     13703   75,00   27,00    0,00    0,00  100,00     3  node
04:31:59 PM  1000     13703   73,00   30,00    0,00    0,00  100,00     3  node
04:32:00 PM  1000     13703   68,00   33,00    0,00    0,00  100,00     0  node
04:32:01 PM  1000     13703   77,00   25,00    0,00    0,00  100,00     2  node
04:32:02 PM  1000     13703   72,00   29,00    0,00    0,00  100,00     0  node
04:32:03 PM  1000     13703   66,00   35,00    0,00    0,00  100,00     3  node
04:32:04 PM  1000     13703   78,00   25,00    0,00    0,00  100,00     3  node
04:32:05 PM  1000     13703   72,00   29,00    0,00    0,00  100,00     0  node
04:32:06 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     3  node
04:32:07 PM  1000     13703   70,00   32,00    0,00    0,00  100,00     2  node
04:32:08 PM  1000     13703   70,00   31,00    0,00    0,00  100,00     0  node
04:32:09 PM  1000     13703   66,00   35,00    0,00    1,00  100,00     0  node
04:32:10 PM  1000     13703   69,00   33,00    0,00    0,00  100,00     0  node
04:32:11 PM  1000     13703   77,00   25,00    0,00    0,00  100,00     2  node
04:32:12 PM  1000     13703   71,00   30,00    0,00    0,00  100,00     3  node
04:32:13 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     3  node
04:32:14 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     2  node
04:32:15 PM  1000     13703   68,00   33,00    0,00    0,00  100,00     2  node
04:32:16 PM  1000     13703   76,00   26,00    0,00    0,00  100,00     2  node
04:32:17 PM  1000     13703   71,00   31,00    0,00    1,00  100,00     0  node
04:32:18 PM  1000     13703   70,00   32,00    0,00    0,00  100,00     1  node
04:32:19 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     2  node
04:32:20 PM  1000     13703   69,00   31,00    0,00    0,00  100,00     3  node
04:32:21 PM  1000     13703   74,00   29,00    0,00    0,00  100,00     1  node
04:32:22 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     2  node
04:32:23 PM  1000     13703   76,00   25,00    0,00    0,00  100,00     3  node
04:32:24 PM  1000     13703   75,00   27,00    0,00    0,00  100,00     2  node
04:32:25 PM  1000     13703   69,00   33,00    0,00    0,00  100,00     3  node
04:32:26 PM  1000     13703   70,00   32,00    0,00    0,00  100,00     3  node
04:32:27 PM  1000     13703   67,00   34,00    0,00    0,00  100,00     3  node
04:32:28 PM  1000     13703   70,00   32,00    0,00    0,00  100,00     0  node
04:32:29 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     3  node
04:32:30 PM  1000     13703   79,00   22,00    0,00    0,00  100,00     0  node
04:32:31 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     0  node
04:32:32 PM  1000     13703   78,00   24,00    0,00    0,00  100,00     2  node
04:32:33 PM  1000     13703   69,00   33,00    0,00    0,00  100,00     1  node
04:32:34 PM  1000     13703   76,00   26,00    0,00    0,00  100,00     2  node
04:32:35 PM  1000     13703   67,00   34,00    0,00    0,00  100,00     2  node
04:32:36 PM  1000     13703   68,00   34,00    0,00    0,00  100,00     0  node
04:32:37 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     1  node
04:32:38 PM  1000     13703   75,00   26,00    0,00    0,00  100,00     1  node
04:32:39 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     2  node
04:32:40 PM  1000     13703   69,00   32,00    0,00    0,00  100,00     0  node
04:32:41 PM  1000     13703   72,00   31,00    0,00    0,00  100,00     0  node
04:32:42 PM  1000     13703   70,00   31,00    0,00    0,00  100,00     2  node
04:32:43 PM  1000     13703   78,00   24,00    0,00    0,00  100,00     0  node
04:32:44 PM  1000     13703   72,00   29,00    0,00    0,00  100,00     2  node
04:32:45 PM  1000     13703   70,00   33,00    0,00    1,00  100,00     1  node
04:32:46 PM  1000     13703   72,00   29,00    0,00    0,00  100,00     2  node
04:32:47 PM  1000     13703   71,00   30,00    0,00    0,00  100,00     1  node
04:32:48 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     3  node
04:32:49 PM  1000     13703   68,00   34,00    0,00    0,00  100,00     0  node
04:32:50 PM  1000     13703   75,00   26,00    0,00    0,00  100,00     3  node
04:32:51 PM  1000     13703   77,00   25,00    0,00    0,00  100,00     1  node
04:32:52 PM  1000     13703   69,00   32,00    0,00    0,00  100,00     1  node
04:32:53 PM  1000     13703   15,00    6,00    0,00    0,00   21,00     0  node
04:32:54 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     0  node
04:32:55 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     0  node
04:32:56 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     0  node
04:32:57 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     0  node
04:32:58 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     0  node
04:32:59 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     0  node
04:33:00 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     0  node
04:33:01 PM  1000     13703   51,00   28,00    0,00    0,00   79,00     3  node
04:33:02 PM  1000     13703   68,00   34,00    0,00    0,00  100,00     3  node
04:33:03 PM  1000     13703   66,00   36,00    0,00    0,00  100,00     3  node
04:33:04 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     3  node
04:33:05 PM  1000     13703   75,00   27,00    0,00    0,00  100,00     1  node
04:33:06 PM  1000     13703   70,00   31,00    0,00    0,00  100,00     3  node
04:33:07 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     3  node
04:33:08 PM  1000     13703   68,00   34,00    0,00    0,00  100,00     3  node
04:33:09 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     3  node
04:33:10 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     3  node
04:33:11 PM  1000     13703   75,00   25,00    0,00    0,00  100,00     0  node
04:33:12 PM  1000     13703   75,00   27,00    0,00    0,00  100,00     0  node
04:33:13 PM  1000     13703   76,00   26,00    0,00    0,00  100,00     2  node
04:33:14 PM  1000     13703   71,00   32,00    0,00    0,00  100,00     0  node
04:33:15 PM  1000     13703   75,00   26,00    0,00    0,00  100,00     1  node
04:33:16 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     3  node
04:33:17 PM  1000     13703   73,00   28,00    0,00    0,00  100,00     3  node
04:33:18 PM  1000     13703   78,00   24,00    0,00    0,00  100,00     1  node
04:33:19 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     1  node
04:33:20 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     1  node
04:33:21 PM  1000     13703   72,00   29,00    0,00    0,00  100,00     1  node
04:33:22 PM  1000     13703   73,00   29,00    0,00    0,00  100,00     1  node
04:33:23 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     1  node
04:33:24 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     0  node
04:33:25 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     2  node
04:33:26 PM  1000     13703   62,00   40,00    0,00    0,00  100,00     0  node
04:33:27 PM  1000     13703   65,00   37,00    0,00    0,00  100,00     0  node
04:33:28 PM  1000     13703   68,00   33,00    0,00    0,00  100,00     0  node
04:33:29 PM  1000     13703   64,00   38,00    0,00    1,00  100,00     2  node
04:33:30 PM  1000     13703   69,00   33,00    0,00    0,00  100,00     2  node
04:33:31 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     0  node
04:33:32 PM  1000     13703   70,00   31,00    0,00    0,00  100,00     1  node
04:33:33 PM  1000     13703   69,00   33,00    0,00    0,00  100,00     1  node
04:33:34 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     1  node
04:33:35 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     3  node
04:33:36 PM  1000     13703   73,00   28,00    0,00    0,00  100,00     2  node
04:33:37 PM  1000     13703   70,00   32,00    0,00    0,00  100,00     0  node
04:33:38 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     0  node
04:33:39 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     0  node
04:33:40 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     0  node
04:33:41 PM  1000     13703   73,00   29,00    0,00    0,00  100,00     0  node
04:33:42 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     0  node
04:33:43 PM  1000     13703   74,00   28,00    0,00    0,00  100,00     0  node
04:33:44 PM  1000     13703   70,00   32,00    0,00    0,00  100,00     2  node
04:33:45 PM  1000     13703   76,00   26,00    0,00    0,00  100,00     1  node
04:33:46 PM  1000     13703   69,00   32,00    0,00    0,00  100,00     1  node
04:33:47 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     3  node
04:33:48 PM  1000     13703   71,00   31,00    0,00    0,00  100,00     3  node
04:33:49 PM  1000     13703   77,00   24,00    0,00    0,00  100,00     3  node
04:33:50 PM  1000     13703   72,00   30,00    0,00    0,00  100,00     3  node
04:33:51 PM  1000     13703   71,00   30,00    0,00    0,00  100,00     3  node
04:33:52 PM  1000     13703   73,00   29,00    0,00    0,00  100,00     1  node
04:33:53 PM  1000     13703   73,00   30,00    0,00    0,00  100,00     1  node
04:33:54 PM  1000     13703   65,00   36,00    0,00    0,00  100,00     1  node
04:33:55 PM  1000     13703   68,00   34,00    0,00    0,00  100,00     1  node
04:33:56 PM  1000     13703   71,00   30,00    0,00    0,00  100,00     1  node
04:33:57 PM  1000     13703   68,00   34,00    0,00    0,00  100,00     3  node
04:33:58 PM  1000     13703   64,00   26,00    0,00    0,00   90,00     1  node
04:33:59 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     1  node
04:34:00 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     1  node
04:34:01 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     1  node
04:34:02 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     1  node
04:34:03 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     1  node
04:34:04 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     1  node
04:34:05 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     1  node
04:34:06 PM  1000     13703    0,00    0,00    0,00    0,00    0,00     1  node

Average:     1000     13703   57,40   24,09    0,00    0,03   81,49     -  node

```
{% endcapture %}
{% include widgets/toggle-panel.html toggle-name="nodejs-puro-pidstats-toggle" button-text="NodeJS Puro" toggle-text=nodejs-pure-pidstats  footer="Fim do pidstats do NodeJS Puro" %}


{% capture java-spring-webflux-pidstats %}
```
05:16:08 PM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
05:16:09 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:16:10 PM  1000     15740    0,00    1,00    0,00    0,00    1,00     2  java
05:16:11 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:16:12 PM  1000     15740    0,00    1,00    0,00    0,00    1,00     2  java
05:16:13 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:16:14 PM  1000     15740    0,00    0,00    0,00    0,00    0,00     2  java
05:16:15 PM  1000     15740    0,00    1,00    0,00    0,00    1,00     2  java
05:16:16 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:16:17 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:16:18 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:16:19 PM  1000     15740    1,00    1,00    0,00    0,00    2,00     2  java
05:16:20 PM  1000     15740    2,00    0,00    0,00    0,00    2,00     2  java
05:16:21 PM  1000     15740   31,00    5,00    0,00    0,00   36,00     2  java
05:16:22 PM  1000     15740  100,00   61,00    0,00    0,00  100,00     2  java
05:16:23 PM  1000     15740  100,00   75,00    0,00    0,00  100,00     2  java
05:16:24 PM  1000     15740  100,00   71,00    0,00    0,00  100,00     2  java
05:16:25 PM  1000     15740  100,00   79,00    0,00    0,00  100,00     2  java
05:16:26 PM  1000     15740  100,00   68,00    0,00    0,00  100,00     2  java
05:16:27 PM  1000     15740  100,00   72,00    0,00    0,00  100,00     2  java
05:16:28 PM  1000     15740  100,00   79,00    0,00    0,00  100,00     2  java
05:16:29 PM  1000     15740  100,00   68,00    0,00    0,00  100,00     2  java
05:16:30 PM  1000     15740  100,00   73,00    0,00    0,00  100,00     2  java
05:16:31 PM  1000     15740  100,00   72,00    0,00    0,00  100,00     2  java
05:16:32 PM  1000     15740  100,00   73,00    0,00    0,00  100,00     2  java
05:16:33 PM  1000     15740  100,00   72,00    0,00    0,00  100,00     2  java
05:16:34 PM  1000     15740  100,00   73,00    0,00    0,00  100,00     2  java
05:16:35 PM  1000     15740  100,00   72,00    0,00    0,00  100,00     2  java
05:16:36 PM  1000     15740  100,00   78,00    0,00    0,00  100,00     2  java
05:16:37 PM  1000     15740  100,00   74,00    0,00    0,00  100,00     2  java
05:16:38 PM  1000     15740  100,00   71,00    0,00    0,00  100,00     2  java
05:16:39 PM  1000     15740  100,00   81,00    0,00    0,00  100,00     2  java
05:16:40 PM  1000     15740  100,00   76,00    0,00    0,00  100,00     2  java
05:16:41 PM  1000     15740  100,00   82,00    0,00    0,00  100,00     2  java
05:16:42 PM  1000     15740  100,00   77,00    0,00    0,00  100,00     2  java
05:16:43 PM  1000     15740  100,00   70,00    0,00    0,00  100,00     2  java
05:16:44 PM  1000     15740  100,00   89,00    0,00    0,00  100,00     2  java
05:16:45 PM  1000     15740  100,00   84,00    0,00    0,00  100,00     2  java
05:16:46 PM  1000     15740  100,00   74,00    0,00    0,00  100,00     2  java
05:16:47 PM  1000     15740  100,00   75,00    0,00    0,00  100,00     2  java
05:16:48 PM  1000     15740  100,00   68,00    0,00    0,00  100,00     2  java
05:16:49 PM  1000     15740  100,00   71,00    0,00    0,00  100,00     2  java
05:16:50 PM  1000     15740  100,00   76,00    0,00    0,00  100,00     2  java
05:16:51 PM  1000     15740  100,00   79,00    0,00    0,00  100,00     2  java
05:16:52 PM  1000     15740  100,00   78,00    0,00    0,00  100,00     2  java
05:16:53 PM  1000     15740  100,00   77,00    0,00    0,00  100,00     2  java
05:16:54 PM  1000     15740  100,00   65,00    0,00    0,00  100,00     2  java
05:16:55 PM  1000     15740  100,00   73,00    0,00    0,00  100,00     2  java
05:16:56 PM  1000     15740  100,00   67,00    0,00    0,00  100,00     2  java
05:16:57 PM  1000     15740  100,00   70,00    0,00    0,00  100,00     2  java
05:16:58 PM  1000     15740  100,00   73,00    0,00    0,00  100,00     2  java
05:16:59 PM  1000     15740  100,00   71,00    0,00    0,00  100,00     2  java
05:17:00 PM  1000     15740  100,00   74,00    0,00    0,00  100,00     2  java
05:17:01 PM  1000     15740  100,00   69,00    0,00    0,00  100,00     2  java
05:17:02 PM  1000     15740  100,00   86,00    0,00    0,00  100,00     2  java
05:17:03 PM  1000     15740  100,00   72,00    0,00    0,00  100,00     2  java
05:17:04 PM  1000     15740  100,00   71,00    0,00    0,00  100,00     2  java
05:17:05 PM  1000     15740  100,00   68,00    0,00    0,00  100,00     2  java
05:17:06 PM  1000     15740  100,00   65,00    0,00    0,00  100,00     2  java
05:17:07 PM  1000     15740  100,00   69,00    0,00    0,00  100,00     2  java
05:17:08 PM  1000     15740  100,00   68,00    0,00    0,00  100,00     2  java
05:17:09 PM  1000     15740  100,00   72,00    0,00    0,00  100,00     2  java
05:17:10 PM  1000     15740  100,00   72,00    0,00    0,00  100,00     2  java
05:17:11 PM  1000     15740  100,00   70,00    0,00    0,00  100,00     2  java
05:17:12 PM  1000     15740  100,00   71,00    0,00    0,00  100,00     2  java
05:17:13 PM  1000     15740  100,00   70,00    0,00    0,00  100,00     2  java
05:17:14 PM  1000     15740  100,00   79,00    0,00    0,00  100,00     2  java
05:17:15 PM  1000     15740  100,00   77,00    0,00    0,00  100,00     2  java
05:17:16 PM  1000     15740  100,00   76,00    0,00    0,00  100,00     2  java
05:17:17 PM  1000     15740  100,00   76,00    0,00    0,00  100,00     2  java
05:17:18 PM  1000     15740  100,00   80,00    0,00    0,00  100,00     2  java
05:17:19 PM  1000     15740  100,00   68,00    0,00    0,00  100,00     2  java
05:17:20 PM  1000     15740  100,00   72,00    0,00    0,00  100,00     2  java
05:17:21 PM  1000     15740  100,00   69,00    0,00    0,00  100,00     2  java
05:17:22 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:17:23 PM  1000     15740    0,00    0,00    0,00    0,00    0,00     2  java
05:17:24 PM  1000     15740    0,00    1,00    0,00    0,00    1,00     2  java
05:17:25 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:17:26 PM  1000     15740    0,00    0,00    0,00    0,00    0,00     2  java
05:17:27 PM  1000     15740    0,00    1,00    0,00    0,00    1,00     2  java
05:17:28 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:17:29 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:17:30 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:17:31 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:17:32 PM  1000     15740  100,00   14,00    0,00    0,00  100,00     2  java
05:17:33 PM  1000     15740   79,00   24,00    0,00    0,00  100,00     2  java
05:17:34 PM  1000     15740   78,00   26,00    0,00    0,00  100,00     2  java
05:17:35 PM  1000     15740   80,00   24,00    0,00    0,00  100,00     2  java
05:17:36 PM  1000     15740   75,00   28,00    0,00    0,00  100,00     2  java
05:17:37 PM  1000     15740   79,00   33,00    0,00    0,00  100,00     2  java
05:17:38 PM  1000     15740   85,00   20,00    0,00    0,00  100,00     2  java
05:17:39 PM  1000     15740   75,00   27,00    0,00    0,00  100,00     2  java
05:17:40 PM  1000     15740   79,00   24,00    0,00    0,00  100,00     2  java
05:17:41 PM  1000     15740   77,00   25,00    0,00    0,00  100,00     2  java
05:17:42 PM  1000     15740   77,00   27,00    0,00    0,00  100,00     2  java
05:17:43 PM  1000     15740   77,00   26,00    0,00    0,00  100,00     2  java
05:17:44 PM  1000     15740   73,00   30,00    0,00    0,00  100,00     2  java
05:17:45 PM  1000     15740   71,00   32,00    0,00    0,00  100,00     2  java
05:17:46 PM  1000     15740   77,00   27,00    0,00    0,00  100,00     2  java
05:17:47 PM  1000     15740   72,00   32,00    0,00    0,00  100,00     2  java
05:17:48 PM  1000     15740   76,00   27,00    0,00    0,00  100,00     2  java
05:17:49 PM  1000     15740   78,00   25,00    0,00    0,00  100,00     2  java
05:17:50 PM  1000     15740   81,00   25,00    0,00    0,00  100,00     2  java
05:17:51 PM  1000     15740   76,00   27,00    0,00    0,00  100,00     2  java
05:17:52 PM  1000     15740   75,00   27,00    0,00    0,00  100,00     2  java
05:17:53 PM  1000     15740   73,00   29,00    0,00    0,00  100,00     2  java
05:17:54 PM  1000     15740   75,00   28,00    0,00    0,00  100,00     2  java
05:17:55 PM  1000     15740   70,00   34,00    0,00    0,00  100,00     2  java
05:17:56 PM  1000     15740   78,00   26,00    0,00    0,00  100,00     2  java
05:17:57 PM  1000     15740   76,00   28,00    0,00    0,00  100,00     2  java
05:17:58 PM  1000     15740   80,00   24,00    0,00    0,00  100,00     2  java
05:17:59 PM  1000     15740   82,00   25,00    0,00    0,00  100,00     2  java
05:18:00 PM  1000     15740   78,00   28,00    0,00    0,00  100,00     2  java
05:18:01 PM  1000     15740   72,00   32,00    0,00    0,00  100,00     2  java
05:18:02 PM  1000     15740   79,00   24,00    0,00    0,00  100,00     2  java
05:18:03 PM  1000     15740   76,00   26,00    0,00    0,00  100,00     2  java
05:18:04 PM  1000     15740   79,00   29,00    0,00    0,00  100,00     2  java
05:18:05 PM  1000     15740   73,00   30,00    0,00    0,00  100,00     2  java
05:18:06 PM  1000     15740   78,00   26,00    0,00    0,00  100,00     2  java
05:18:07 PM  1000     15740   81,00   23,00    0,00    0,00  100,00     2  java
05:18:08 PM  1000     15740   78,00   27,00    0,00    0,00  100,00     2  java
05:18:09 PM  1000     15740   84,00   23,00    0,00    0,00  100,00     2  java
05:18:10 PM  1000     15740   77,00   25,00    0,00    0,00  100,00     2  java
05:18:11 PM  1000     15740   84,00   23,00    0,00    0,00  100,00     2  java
05:18:12 PM  1000     15740   80,00   25,00    0,00    0,00  100,00     2  java
05:18:13 PM  1000     15740   78,00   27,00    0,00    0,00  100,00     2  java
05:18:14 PM  1000     15740   77,00   25,00    0,00    0,00  100,00     2  java
05:18:15 PM  1000     15740   76,00   30,00    0,00    0,00  100,00     2  java
05:18:16 PM  1000     15740   76,00   26,00    0,00    0,00  100,00     2  java
05:18:17 PM  1000     15740   78,00   27,00    0,00    0,00  100,00     2  java
05:18:18 PM  1000     15740   76,00   28,00    0,00    0,00  100,00     2  java
05:18:19 PM  1000     15740   77,00   23,00    0,00    0,00  100,00     2  java
05:18:20 PM  1000     15740   79,00   25,00    0,00    0,00  100,00     2  java
05:18:21 PM  1000     15740   76,00   27,00    0,00    0,00  100,00     2  java
05:18:22 PM  1000     15740   79,00   29,00    0,00    0,00  100,00     2  java
05:18:23 PM  1000     15740   75,00   29,00    0,00    0,00  100,00     2  java
05:18:24 PM  1000     15740   75,00   29,00    0,00    0,00  100,00     2  java
05:18:25 PM  1000     15740   77,00   26,00    0,00    0,00  100,00     2  java
05:18:26 PM  1000     15740   79,00   25,00    0,00    0,00  100,00     2  java
05:18:27 PM  1000     15740   75,00   27,00    0,00    0,00  100,00     2  java
05:18:28 PM  1000     15740   78,00   26,00    0,00    0,00  100,00     2  java
05:18:29 PM  1000     15740   90,00   23,00    0,00    0,00  100,00     2  java
05:18:30 PM  1000     15740   80,00   23,00    0,00    0,00  100,00     2  java
05:18:31 PM  1000     15740   80,00   28,00    0,00    0,00  100,00     2  java
05:18:32 PM  1000     15740   75,00   28,00    0,00    0,00  100,00     2  java
05:18:33 PM  1000     15740   74,00   29,00    0,00    0,00  100,00     2  java
05:18:34 PM  1000     15740   86,00   22,00    0,00    0,00  100,00     2  java
05:18:35 PM  1000     15740   77,00   27,00    0,00    0,00  100,00     2  java
05:18:36 PM  1000     15740   74,00   28,00    0,00    0,00  100,00     2  java
05:18:37 PM  1000     15740   75,00   31,00    0,00    0,00  100,00     2  java
05:18:38 PM  1000     15740   73,00   30,00    0,00    0,00  100,00     2  java
05:18:39 PM  1000     15740   79,00   27,00    0,00    0,00  100,00     2  java
05:18:40 PM  1000     15740   78,00   26,00    0,00    0,00  100,00     2  java
05:18:41 PM  1000     15740   78,00   26,00    0,00    0,00  100,00     2  java
05:18:42 PM  1000     15740   76,00   26,00    0,00    0,00  100,00     2  java
05:18:43 PM  1000     15740   77,00   28,00    0,00    0,00  100,00     2  java
05:18:44 PM  1000     15740   72,00   32,00    0,00    0,00  100,00     2  java
05:18:45 PM  1000     15740   71,00   33,00    0,00    0,00  100,00     2  java
05:18:46 PM  1000     15740   79,21   24,75    0,00    0,00  100,00     2  java
05:18:47 PM  1000     15740   79,00   26,00    0,00    0,00  100,00     2  java
05:18:48 PM  1000     15740   75,00   30,00    0,00    0,00  100,00     2  java
05:18:49 PM  1000     15740   80,00   24,00    0,00    0,00  100,00     2  java
05:18:50 PM  1000     15740   78,00   26,00    0,00    0,00  100,00     2  java
05:18:51 PM  1000     15740   77,00   28,00    0,00    0,00  100,00     2  java
05:18:52 PM  1000     15740   86,00   22,00    0,00    0,00  100,00     2  java
05:18:53 PM  1000     15740   81,00   28,00    0,00    0,00  100,00     2  java
05:18:54 PM  1000     15740   73,00   30,00    0,00    0,00  100,00     2  java
05:18:55 PM  1000     15740   79,00   23,00    0,00    0,00  100,00     2  java
05:18:56 PM  1000     15740   81,00   24,00    0,00    0,00  100,00     2  java
05:18:57 PM  1000     15740   78,00   28,00    0,00    0,00  100,00     2  java
05:18:58 PM  1000     15740   77,00   28,00    0,00    0,00  100,00     2  java
05:18:59 PM  1000     15740   79,00   24,00    0,00    0,00  100,00     2  java
05:19:00 PM  1000     15740   83,00   22,00    0,00    0,00  100,00     2  java
05:19:01 PM  1000     15740   77,00   26,00    0,00    0,00  100,00     2  java
05:19:02 PM  1000     15740   82,00   26,00    0,00    0,00  100,00     2  java
05:19:03 PM  1000     15740   83,00   30,00    0,00    0,00  100,00     2  java
05:19:04 PM  1000     15740   76,00   28,00    0,00    0,00  100,00     2  java
05:19:05 PM  1000     15740   77,00   27,00    0,00    0,00  100,00     2  java
05:19:06 PM  1000     15740   81,00   23,00    0,00    0,00  100,00     2  java
05:19:07 PM  1000     15740   74,00   29,00    0,00    0,00  100,00     2  java
05:19:08 PM  1000     15740   81,00   27,00    0,00    0,00  100,00     2  java
05:19:09 PM  1000     15740   75,00   29,00    0,00    0,00  100,00     2  java
05:19:10 PM  1000     15740   82,00   24,00    0,00    0,00  100,00     2  java
05:19:11 PM  1000     15740   89,00   29,00    0,00    0,00  100,00     2  java
05:19:12 PM  1000     15740   80,00   24,00    0,00    0,00  100,00     2  java
05:19:13 PM  1000     15740   75,00   27,00    0,00    0,00  100,00     2  java
05:19:14 PM  1000     15740   84,00   26,00    0,00    0,00  100,00     2  java
05:19:15 PM  1000     15740   80,00   24,00    0,00    0,00  100,00     2  java
05:19:16 PM  1000     15740   78,00   26,00    0,00    0,00  100,00     2  java
05:19:17 PM  1000     15740   81,00   26,00    0,00    0,00  100,00     2  java
05:19:18 PM  1000     15740   85,00   19,00    0,00    0,00  100,00     2  java
05:19:19 PM  1000     15740   77,00   27,00    0,00    0,00  100,00     2  java
05:19:20 PM  1000     15740   78,00   29,00    0,00    0,00  100,00     2  java
05:19:21 PM  1000     15740   65,00   19,00    0,00    0,00   84,00     2  java
05:19:22 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:19:23 PM  1000     15740    0,00    0,00    0,00    0,00    0,00     2  java
05:19:24 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:19:25 PM  1000     15740    0,00    0,00    0,00    0,00    0,00     2  java
05:19:26 PM  1000     15740    1,00    0,00    0,00    0,00    1,00     2  java
05:19:27 PM  1000     15740    0,00    1,00    0,00    0,00    1,00     2  java

Average:     1000     15740  100,00   36,82    0,00    0,00  100,00     -  java

```
{% endcapture %}
{% include widgets/toggle-panel.html toggle-name="java-spring-webflux-toggle" button-text="Java Spring Webflux" toggle-text=java-spring-webflux-pidstats  footer="Fim do pidstats do Java Spring Webflux" %}


{% capture java-vertx-pidstats %}
```
05:37:40 PM   UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
05:37:41 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:37:42 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:37:43 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:37:44 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:37:45 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:37:46 PM  1000     16395    0,00    1,00    0,00    0,00    1,00     3  java
05:37:47 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:37:48 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:37:49 PM  1000     16395   45,00    9,00    0,00    0,00   54,00     3  java
05:37:50 PM  1000     16395   59,00   47,00    0,00    0,00  100,00     3  java
05:37:51 PM  1000     16395   56,00   46,00    0,00    0,00  100,00     3  java
05:37:52 PM  1000     16395   56,00   48,00    0,00    0,00  100,00     3  java
05:37:53 PM  1000     16395   58,00   45,00    0,00    0,00  100,00     3  java
05:37:54 PM  1000     16395   55,00   48,00    0,00    0,00  100,00     3  java
05:37:55 PM  1000     16395   53,00   51,00    0,00    0,00  100,00     3  java
05:37:56 PM  1000     16395   63,00   47,00    0,00    0,00  100,00     3  java
05:37:57 PM  1000     16395   54,00   49,00    0,00    0,00  100,00     3  java
05:37:58 PM  1000     16395   57,00   46,00    0,00    0,00  100,00     3  java
05:37:59 PM  1000     16395   54,00   49,00    0,00    0,00  100,00     3  java
05:38:00 PM  1000     16395   51,00   51,00    0,00    0,00  100,00     3  java
05:38:01 PM  1000     16395   46,00   58,00    0,00    0,00  100,00     3  java
05:38:02 PM  1000     16395   53,00   49,00    0,00    0,00  100,00     3  java
05:38:03 PM  1000     16395   55,00   48,00    0,00    0,00  100,00     3  java
05:38:04 PM  1000     16395   48,00   55,00    0,00    0,00  100,00     3  java
05:38:05 PM  1000     16395   46,00   52,00    0,00    0,00   98,00     3  java
05:38:06 PM  1000     16395   51,00   44,00    0,00    0,00   95,00     3  java
05:38:07 PM  1000     16395   53,00   49,00    0,00    0,00  100,00     3  java
05:38:08 PM  1000     16395   53,00   50,00    0,00    0,00  100,00     3  java
05:38:09 PM  1000     16395   48,00   55,00    0,00    0,00  100,00     3  java
05:38:10 PM  1000     16395   52,00   51,00    0,00    0,00  100,00     3  java
05:38:11 PM  1000     16395   50,00   53,00    0,00    0,00  100,00     3  java
05:38:12 PM  1000     16395   47,00   53,00    0,00    0,00  100,00     3  java
05:38:13 PM  1000     16395   48,00   54,00    0,00    0,00  100,00     3  java
05:38:14 PM  1000     16395   54,00   48,00    0,00    0,00  100,00     3  java
05:38:15 PM  1000     16395   55,00   48,00    0,00    0,00  100,00     3  java
05:38:16 PM  1000     16395   56,00   47,00    0,00    0,00  100,00     3  java
05:38:17 PM  1000     16395   55,00   49,00    0,00    0,00  100,00     3  java
05:38:18 PM  1000     16395   50,00   52,00    0,00    0,00  100,00     3  java
05:38:19 PM  1000     16395   56,00   47,00    0,00    0,00  100,00     3  java
05:38:20 PM  1000     16395   54,00   49,00    0,00    0,00  100,00     3  java
05:38:21 PM  1000     16395   52,00   51,00    0,00    0,00  100,00     3  java
05:38:22 PM  1000     16395   55,00   48,00    0,00    0,00  100,00     3  java
05:38:23 PM  1000     16395   50,00   53,00    0,00    0,00  100,00     3  java
05:38:24 PM  1000     16395   50,00   53,00    0,00    0,00  100,00     3  java
05:38:25 PM  1000     16395   54,00   50,00    0,00    0,00  100,00     3  java
05:38:26 PM  1000     16395   53,00   49,00    0,00    0,00  100,00     3  java
05:38:27 PM  1000     16395   57,00   46,00    0,00    0,00  100,00     3  java
05:38:28 PM  1000     16395   47,00   55,00    0,00    0,00  100,00     3  java
05:38:29 PM  1000     16395   54,00   50,00    0,00    0,00  100,00     3  java
05:38:30 PM  1000     16395   57,00   45,00    0,00    0,00  100,00     3  java
05:38:31 PM  1000     16395   56,00   48,00    0,00    0,00  100,00     3  java
05:38:32 PM  1000     16395   55,00   47,00    0,00    0,00  100,00     3  java
05:38:33 PM  1000     16395   60,00   44,00    0,00    0,00  100,00     3  java
05:38:34 PM  1000     16395   52,00   51,00    0,00    0,00  100,00     3  java
05:38:35 PM  1000     16395   58,00   44,00    0,00    0,00  100,00     3  java
05:38:36 PM  1000     16395   57,00   47,00    0,00    0,00  100,00     3  java
05:38:37 PM  1000     16395   53,00   50,00    0,00    0,00  100,00     3  java
05:38:38 PM  1000     16395   51,00   52,00    0,00    0,00  100,00     3  java
05:38:39 PM  1000     16395   49,00   54,00    0,00    0,00  100,00     3  java
05:38:40 PM  1000     16395   57,00   45,00    0,00    0,00  100,00     3  java
05:38:41 PM  1000     16395   58,00   45,00    0,00    0,00  100,00     3  java
05:38:42 PM  1000     16395   52,00   51,00    0,00    0,00  100,00     3  java
05:38:43 PM  1000     16395   53,00   50,00    0,00    0,00  100,00     3  java
05:38:44 PM  1000     16395   56,00   47,00    0,00    0,00  100,00     3  java
05:38:45 PM  1000     16395   55,00   48,00    0,00    0,00  100,00     3  java
05:38:46 PM  1000     16395   52,00   50,00    0,00    0,00  100,00     3  java
05:38:47 PM  1000     16395   57,00   46,00    0,00    0,00  100,00     3  java
05:38:48 PM  1000     16395   54,00   49,00    0,00    0,00  100,00     3  java
05:38:49 PM  1000     16395   53,00   44,00    0,00    0,00   97,00     3  java
05:38:50 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:38:51 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:38:52 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:38:53 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:38:54 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:38:55 PM  1000     16395    0,00    1,00    0,00    0,00    1,00     3  java
05:38:56 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:38:57 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:38:58 PM  1000     16395    1,00    0,00    0,00    0,00    1,00     3  java
05:38:59 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:39:00 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:39:01 PM  1000     16395   49,00   25,00    0,00    0,00   74,00     3  java
05:39:02 PM  1000     16395   48,00   39,00    0,00    0,00   87,00     3  java
05:39:03 PM  1000     16395   49,00   37,00    0,00    0,00   86,00     3  java
05:39:04 PM  1000     16395   47,00   40,00    0,00    0,00   87,00     3  java
05:39:05 PM  1000     16395   47,00   40,00    0,00    0,00   87,00     3  java
05:39:06 PM  1000     16395   42,00   44,00    0,00    0,00   86,00     3  java
05:39:07 PM  1000     16395   43,00   44,00    0,00    0,00   87,00     3  java
05:39:08 PM  1000     16395   44,00   44,00    0,00    0,00   88,00     3  java
05:39:09 PM  1000     16395   66,00   33,00    0,00    0,00   99,00     3  java
05:39:10 PM  1000     16395   53,00   32,00    0,00    0,00   85,00     3  java
05:39:11 PM  1000     16395   48,00   40,00    0,00    0,00   88,00     3  java
05:39:12 PM  1000     16395   46,00   41,00    0,00    0,00   87,00     3  java
05:39:13 PM  1000     16395   46,00   40,00    0,00    0,00   86,00     3  java
05:39:14 PM  1000     16395   51,00   36,00    0,00    0,00   87,00     3  java
05:39:15 PM  1000     16395   42,00   44,00    0,00    0,00   86,00     3  java
05:39:16 PM  1000     16395   53,00   34,00    0,00    0,00   87,00     3  java
05:39:17 PM  1000     16395   45,00   40,00    0,00    0,00   85,00     3  java
05:39:18 PM  1000     16395   44,00   41,00    0,00    0,00   85,00     3  java
05:39:19 PM  1000     16395   50,00   37,00    0,00    0,00   87,00     3  java
05:39:20 PM  1000     16395   50,00   37,00    0,00    0,00   87,00     3  java
05:39:21 PM  1000     16395   49,00   37,00    0,00    0,00   86,00     3  java
05:39:22 PM  1000     16395   47,00   40,00    0,00    0,00   87,00     3  java
05:39:23 PM  1000     16395   48,00   36,00    0,00    0,00   84,00     3  java
05:39:24 PM  1000     16395   48,00   38,00    0,00    0,00   86,00     3  java
05:39:25 PM  1000     16395   46,00   41,00    0,00    0,00   87,00     3  java
05:39:26 PM  1000     16395   43,00   42,00    0,00    0,00   85,00     3  java
05:39:27 PM  1000     16395   40,00   47,00    0,00    0,00   87,00     3  java
05:39:28 PM  1000     16395   40,00   45,00    0,00    0,00   85,00     3  java
05:39:29 PM  1000     16395   44,00   42,00    0,00    0,00   86,00     3  java
05:39:30 PM  1000     16395   47,00   41,00    0,00    0,00   88,00     3  java
05:39:31 PM  1000     16395   45,00   40,00    0,00    0,00   85,00     3  java
05:39:32 PM  1000     16395   55,00   32,00    0,00    0,00   87,00     3  java
05:39:33 PM  1000     16395   47,00   39,00    0,00    0,00   86,00     3  java
05:39:34 PM  1000     16395   46,00   41,00    0,00    0,00   87,00     3  java
05:39:35 PM  1000     16395   50,00   37,00    0,00    0,00   87,00     3  java
05:39:36 PM  1000     16395   52,00   34,00    0,00    0,00   86,00     3  java
05:39:37 PM  1000     16395   48,00   38,00    0,00    0,00   86,00     3  java
05:39:38 PM  1000     16395   48,00   37,00    0,00    0,00   85,00     3  java
05:39:39 PM  1000     16395   48,00   39,00    0,00    0,00   87,00     3  java
05:39:40 PM  1000     16395   49,00   36,00    0,00    0,00   85,00     3  java
05:39:41 PM  1000     16395   52,00   34,00    0,00    0,00   86,00     3  java
05:39:42 PM  1000     16395   44,00   41,00    0,00    0,00   85,00     3  java
05:39:43 PM  1000     16395   48,00   38,00    0,00    0,00   86,00     3  java
05:39:44 PM  1000     16395   48,00   35,00    0,00    0,00   83,00     3  java
05:39:45 PM  1000     16395    0,00    0,00    0,00    0,00    0,00     3  java
05:39:46 PM  1000     16395    1,00    0,00    0,00    0,00    1,00     3  java

Average:     1000     16395   42,50   36,97    0,00    0,00   79,47     -  java
``` 
{% endcapture %}
{% include widgets/toggle-panel.html toggle-name="java-vertx-pidstats-toggle" button-text="Java Vert.X" toggle-text=java-vertx-pidstats  footer="Fim do pidstats do Java Vert.X" %}

## Consolidação dos dados
---
Para consolidar os dados, fiz uma tabela com as seguintes informações:

1.  Dos resultados do *wrk*.
    1. Latency Avg. (Linha latency, Coluna Avg) 
    2. Latency Max. (Linha latency, Coluna Max)
    3. Requests/sec

2. Dos resultados do *httperf*
    1. Reply rate avg
    2. Connection time median



| Aplicação | Latency Avg | Latency Max | Requests | Reply rate avg | Connection time median
| -- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| nodejs-express      | 3,62ms | 21,84ms |27.637,66/s|29.665,2/s|673,5ms|
| nodejs-pure         | 3,20ms | 15,90ms |31.283,18/s|34.739,1/s|584,5ms|
| java-spring-webflux | 3,41ms | 56,95ms |34.005,94/s|18.240,9/s|1.091,5ms|
| java-vertx          | 1,47ms | 17,43ms |69.094,37/s|45.974,1/s|431,5ms|

## Conclusão
---
Como eu já havia lido em alguns blogs, a aplicação Node com Express teve o desempenho um pouco inferior à aplicação com NodeJS puro. 

Já a aplicação Java com Vert.x destacou-se em quase todas as métricas escolhidas, porém, a latência máxima ficou um pouco acima da aplicação NodeJS puro. Talvez seja possível melhorar essa métrica (possivelmente em detrimento de outras) regulando parâmetros do Garbage Collector da JVM.

A aplicação Java com Spring Webflux teve o  pior desempenho nesses testes.

Não encontrei dados muito relevantes nas saídas dos comandos *pistat*, portanto não foram incluídos na consolidação. Todas as quatro aplicações usaram praticamente uma 1 CPU inteira.

## Palavras finais 
Como eu já disse anteriormente, esses testes não são e nem tem a pretensão de ser conclusivos. Quem tem um pouco de experiencia com esse tipo de benchmarking sabe que uma pequena alteração na rede, no sistema operacional, na versão do framework ou em configurações, pode alterar totalmente o resultado final.
O principal objetivo desse post é de apresentar os meus testes de forma reproduzível. Portanto, se você gostaria de ver por sí mesmo os resultados, ou melhorar o teste em algum aspecto, as aplicações estão no GitHub e o roteiro de testes se encontra no inicio desse post. Peço apenas que você compartilhe os resultados, pois eu estarei bastante interessado em novas visões sobre esses cenários.

