# Guia prático em containers para iniciantes com DOCKER

## Descrição da atividade

Esta atividade contém instruções passo a passo sobre como começar a usar o Docker.

Aprenderemos como:

- Criar e executar uma imagem como um container
- Compartilhar imagens usando o Docker Hub
- Implantar aplicativos Docker usando vários containers com um banco de dados
- Executar aplicativos usando o Docker Compose

Além disso, você também aprenderá sobre as práticas recomendadas para criar imagens, incluindo instruções sobre como verificar vulnerabilidades de segurança em suas imagens.

### Mas o que é um container?

Simplificando, um container é um processo em área restrita em sua máquina que é isolado de todos os outros processos na máquina servidora.

Esse isolamento aproveita namespaces e cgroups do kernel, recursos que estão no Linux há muito tempo.

O Docker trabalhou para tornar esses recursos acessíveis e fáceis de usar. Para resumir, um container:

- É uma instância executável de uma imagem. Você pode criar, iniciar, parar, mover ou excluir um container usando o DockerAPI ou CLI.
- Pode ser executado em máquinas locais, máquinas virtuais ou implantado na nuvem.
- É portátil (pode ser executado em qualquer sistema operacional).
- Os containers são isolados uns dos outros e executam seus próprios softwares, binários e configurações.

Ao executar um container, ele usa um sistema de arquivos isolado. Esse sistema de arquivos personalizado é fornecido por uma imagem de container.

Como a imagem contém o sistema de arquivos do container, ela deve conter tudo o que é necessário para executar um aplicativo (todas as dependências, configurações, scripts, binários, etc.).

A imagem também contém outras configurações para o container, como variáveis de ambiente, um comando padrão a ser executado, e outros metadados.

## Requisitos necessários

- [Docker Engine](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

Caso não possua instalado estes requisitos pode-se utilizar o [Play with Docker](https://labs.play-with-docker.com/).

## Mãos a massa

Primeiro para questão de afinidades e possuir uma cópia do tutorial do próprio Docker vamos executar:

```shell
docker run -d -p 80:80 docker/getting-started
```

Aqui já demos um primeiro passo, onde executamos um container para ser escutado na porta 80 usando a imagem [docker/getting-started](https://hub.docker.com/r/docker/getting-started).

Nota-se que utilizamos algumas flags ao executar o comando:

- <b>-d</b> executa o container em segundo plano
- <b>-p 80:80</b> mapea a porta do host para a porta 80 no container
- <b>docker/getting-started</b> imagem a ser utilizada

Feito isto, vamos executar mais alguns comandos práticos do dia-a-dia:

#### Exibir containers
```shell
docker container ls
```

#### Logs de um container
```shell
docker logs -f ${id}
```

#### Iniciar um container
```shell
docker start ${id}
```

#### Parar um container
```shell
docker stop ${id}
```

#### Exibir imagens
```shell
docker images
```

#### Exibir detalhes da imagem 
```shell
docker image inspect ${id}
```

#### Renomear uma imagem
```shell
docker image tag ${name} ${new_name}
```

#### Criar um volume
```shell
docker volume create ${name}
```

Volumes de uma forma geral, é a persitência do container.

#### Exibir detalhes de um volume
```shell
docker volume inspect ${name}
```

#### Executar um container com um volume
```shell
docker run -d -v mariadb:/var/lib/mysql mariadb
```

#### Executar comandos dentro de um container
```shell
docker container exec -it ${id} sh
```

- <b>-i</b> interatividade
- <b>-t</b> TTY

#### Criar uma rede
```shell
docker network create ${name}
```

#### Remover um container
```shell
docker rm ${id}
```

#### Remover uma imagem
```shell
docker image rmi ${id}
```

Outro fato que podemos analisar é que ao executar um ```docker run```, pode ser feito de duas formas.

Utilizando ```-it``` o container irá "morrer" ao fechar, e executando com ```-d``` será executado em segundo plano, ou seja, é persistente ao fechar.

## Como criar minha própria imagem?

Para criar sua prórpia imagem, devemos conhecer o ```Dockerfile```, que resumidamente possui os seguintes comandos:

```FROM```: imagem de modelo

```RUN```: executar comandos na criação

```WORKDIR```: diretório padrão/diretório para trabalhar

```COPY```: copiar aplicação

```EXPOSE```: abrir uma porta

```CMD```: parâmetro de inicialização

```ENTRYPOINT```: coração da aplicação

Ao utilizar um ```ENTRYPOINT``` o ```CMD``` fica somente como parâmetros.

Certo, com estes comandos já podemos iniciar nossa própria imagem. Vamos lá!

Baixe os arquivos disponibilizados neste [link](app.zip).

<b>OBS:</b> para quem estiver usando o Play with Docker baixe e arraste e solte o arquivo direto na linha de comando do navegador.

Agora crie um arquivo chamado ```Dockerfile``` no mesmo diretório que se encontra o ```package.json``` da seguinte maneira:

```Dockerfile
FROM node:12-alpine

RUN apk add --no-cache python2 g++ make

WORKDIR /app

COPY . .

RUN yarn install --production

CMD ["node", "src/index.js"]
```

<b>OBS:</b> para quem estiver usando o Play with Docker e deseja editar os aquivos por ali mesmo:

```shell
apk add nano

nano Dockerfile
```

Agora vamos construir esta imagem executando:

```shell
docker build -t getting-started .
```

Ao ser finalizado a construção, iremos iniciar a aplicação executando:

```shell
docker run -dp 3000:3000 getting-started
```

## Preciso atualizar minha aplicação e agora?

Vamos lá, sem problemas que iremos realizar isto.

No arquivo ```src/static/js/app.js```, na linha 56 mude o texto para o seguinte:

```js
<p className="text-center">You have no todo items yet! Add one above!</p>
```

Feito isto, vamos novamente construir nossa imagem executando:

```shell
docker build -t getting-started .
```

Em seguida, executar o container com a imagem atualizada:

```shell
docker run -dp 3000:3000 getting-started
```

# Eba! nosso primeiro erro.

Provalmente você verá um erro semelhante a este:

```shell
docker: Error response from daemon: driver failed programming external connectivity on endpoint vibrant_leavitt (b70795975fe714f88d23fd555c1ec229df2440dfb0e985de2722cba5ceb450ef): Bind for 0.0.0.0:3000 failed: port is already allocated.
```

Isto aconteceu pois estamos tentando criar um container com o nome (getting-started) de um já existente.

Para darmos continuidade, vamos executar os comandos que vimos anteriormente.

```shell
docker ps

docker stop ${id}

docker rm ${id}

docker run -dp 3000:3000 getting-started
```

# Que tal publicarmos nossa imagem?

Ao publicar uma imagem, você estára tornando ela disponível para acesso em vários lugares remotamente, não somente em sua rede.

Você pode publicar a imagem de forma pública, onde todos tem acesso e a outra forma é a privada, onde somente você ou que você convidar terá este acesso.

Certo, nosso primeiro passo é acessar o [Docker Hub](https://hub.docker.com/) e entrar/criar a conta. Após vamos criar nosso primeiro repositório e deixar ele como público mesmo.

Irei criar com o nome de ```getting-started```, mas ele ficará reconhecido como ```${username}/getting-started```, o de vocês ficará com o nome de usuário do Docker Hub.

Nota que no Docker Hub já nos deu um comando para executar:

```shell
docker push ${username}/getting-started
```

Pórem se executarmos irá dar um erro, pois não temos uma imagem com este nome ```${username}/getting-started```:

```shell
Using default tag: latest
The push refers to repository [docker.io/${username}/getting-started]
An image does not exist locally with the tag: ${username}/getting-started
```

Para isto executamos:

```shell
docker tag getting-started ${username}/getting-started
```

Após isto podemos executar novamente, e antes é importante verificarmos se estamos logado, então iremos executar os seguintes comandos:

```shell
docker login

docker push ${username}/getting-started
```

Caso você não informe nenhuma TAG após o nome, ele irá assumir por padrão: ```LATEST```.

# Recapitulando rapidamente

Conseguimos ver alguns comandos básicos do dia-a-dia para utilizarmos o DOCKER, vimos como criar nossa primeira imagem e a como publicar ela para acessar remotamente.

Conseguimos simular alguns erros que possam acontecer e resolver eles.

Mas agora vamos para uma questão, como mantenho esta minha aplicação sem perder os dados quando o container for finalizado?

# Persitências dos dados

Vamos criar um volume, como o mando que vimos anteriormente:

```shell
docker volume create todo-db
```

Após criado, iremos precisar finalizar a aplicação que está em execução com o comando:

```shell
docker rm -f ${id}
```

Após conculsão da exclusão, vamos novamente iniciar nossa aplicação, mas agora com um volume executando:

```shell
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

Pronto, agora podemos até remover a nossa aplicação que nossos dados estarão persistidos. Vamos testar?

# Boas práticas na criação de imagem

O Docker por si só já oferece algumas ferramentas para verficar possíveis vulnerabilidades na imagem, como por exemplo o comando:

```shell
docker scan getting-started
```

Podemos ver também o histório do que foi realizado para criar esta imagem utilizando:

```shell
docker image history getting-started
```

Este comando por si mostrará tudo que foi realizado, podendo encontrar linhas maliciosas caso existam.

# Vamos utilizar o docker-compose?

O docker-compose nos auxilia quando precisamos rodar mais de uma imagem para uma aplicação, como por exemplo criar um site em wordpress com um banco de dados (MySQL).

Para utilizarmos, iremos criar um arquivo com o nome de ```docker-compose.yml``` e neste arquivos iremos escrever:

```yaml
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - "8088:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```

Com ele criado, vamos executar:

```shell
docker-compose up -d
```

Notaram semelhança na hora de rodar? E na sintaxe do yaml?

# Dúvidas?
