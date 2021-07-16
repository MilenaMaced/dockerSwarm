# Docker Swarm


## O que é um Docker Swarm?
Um Docker Swarm é um grupo de máquinas físicas ou virtuais que estão executando o aplicativo Docker e que foram configuradas para se unirem em um cluster. Depois que um grupo de máquinas foi agrupado, você ainda pode executar os comandos do Docker com os quais está acostumado, mas agora eles serão executados pelas máquinas em seu cluster.

Inicializar docker swarm com número do servidor
~~~
$ docker swarm init --advertise-addr “end_ip_servidor”
~~~
Para verificar se swarm está ativo é necessário listar todos os nodes e verificar Availability se nó está ativo.
~~~
$ docker node ls
~~~
Primeira instância manager foi criada e está ativa!
Agora, será necessario criar uma nova instância para cada nó no play docker, para criar os demais nós.

Após a criação das instâncias, vamos verificar se existe algum nó.
~~~
$ docker node ls
~~~
Não existe nenhum nó, agora é necessário acessarmos nossa máquina node 1, para solicitarmos um swarm token para criação de um nó Manager ou um nó work.

Para criação de um nó Manager:
~~~
$ docker swarm join-token manager
~~~
Para criação de um nó Work:
~~~
$ docker swarm join-token worker
~~~

Com a execução de algum dos códigos acima é gerando um token, na qual será necessário a execução deste código na nova instância criada do play docker. Então, seu novo nó foi criado!

Fica ao seu critério a quantidade de criação de nós Manager e Worker. Para essa apresentação será necessária a criacão de três nós worker e um manager. Repita os passos acima até possui todos os 4 nós.


Para visualizar os nós podemos utilizar o visualizer é uma IU da web muito simples que mostra informações básicas sobre nós e contêineres em um swarm do Docker. É um projeto de código aberto no GitHub no dockersamples/docker-swarm-visualizerrepositório.

Visualizer vai rodar na porta 8080
~~~
$ docker run -dit \
-p 8080:8080 \
-v /var/run/docker.sock:/var/run/docker.sock \
dockersamples/visualizer
~~~
Vamos agora para parte mais importante que é a criação de serviços.

## Quais são os dois tipos de serviços do modo Docker Swarm?
O Docker Swarm tem dois tipos de serviços: **replicados e globais**.

**Serviços replicados:** funções de serviços replicados no modo Swarm especificando uma série de tarefas de réplica para o gerenciador de swarm atribuir aos nós disponíveis.

**Serviços globais:** os serviços globais funcionam usando o gerenciador de swam para agendar uma tarefa para cada nó disponível que atenda às restrições de serviços e requisitos de recursos.

### Para criar um serviço replica, rodando a image nginx na porta 4000.


~~~
$ docker service create \
--replicas 1 \
--name replicado -p 4000:80 \
nginx
~~~
Para listar os serviços, basta
~~~
$ docker service ls
~~~
Caso haja necessidade de expansão da aplicação, seja necessário expandir para outros nós, só executar o código abaixo:
~~~
$ docker service scale replicado=3
~~~
Para remover dois
~~~
$ docker service scale replicado=1
~~~

### Para criar um serviço global, rodando a image rayeshuang/friendlyhello na porta 4001, que vai mostrar em qual nó esta rodando serviço.


~~~
$ docker service create \
--mode global \
--name global \
-p 4001:80 \
rayeshuang/friendlyhello
~~~

Para remoção dos serviços, execute a seguinte linha de código com os nomes dos serviços
~~~
$ docker service rm "nomes_dos_servicos"
~~~

#### Para melhor aplicação, vamos criar um serviço mais robusto com a criação do serviço a partir de um arquivo de extensão .yml. Vamos subir os containers com o wordpress e banco de dados mysql, interligados e com sua persistência de dados.

Primeiro é necessário criar as pastas para salvar os arquivos
~~~
$ mkdir /arquivos/bd
~~~
~~~
$ mkdir /arquivos/wordpress
~~~
Criação do arquivo,
~~~
$ vim config.yml
~~~
Basta colocar no vim, o código abaixo:
~~~
version: '3.1'

services:
  db:
    image: mysql:5.7
    restart: always
    deploy:
      replicas: 1
      #mode: global 
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
#    volumes:
#      - /arquivos/bd:/var/lib/mysql

  wordpress:
    image: wordpress
    restart: always
    deploy:
      replicas: 1   
    ports:
      - 4003:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
#    volumes:
#      - /arquivos/wordpress:/var/www/html
~~~
Após fechar o vim, execute:
Para subir o serviço
~~~
docker config deploy -c config.yml wordpress
~~~
Para listar:
~~~
docker config ps wordpress
~~~



