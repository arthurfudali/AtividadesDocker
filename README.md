# AtividadesDocker

# Exercícios

- *Exercícios Docker**

### 🟢 **Fácil**

1. **Rodando um container básico**
    - Execute um container usando a imagem do **Nginx** e acesse a página padrão no navegador.
    - 🔹 *Exemplo de aplicação:* Use a [landing page do TailwindCSS](https://github.com/tailwindtoolbox/Landing-Page) como site estático dentro do container.
    
    Baixar a imagem do nginx:
    
    ```bash
    docker pull nginx
    ```
    
    Criar a pasta com o arquivo index.html dentro da maquina 
    
    ```bash
    mkdir data
    cd data
    mkdir nginx
    cd nginx
    ```
    
    Criar o arquivo `index.html` e copie o código da pagina que ira usar
    
    ```bash
    nano index.html
    ```
    
    Agora inicie o Nginx apontando essa pasta:
    
    ```docker
    docker run --name nginx -d -p 80:80 --volume=/data/nginx:/usr/share/nginx/html nginx
    ```
    
    Use o comando `ip a` para verificar o `ip` do container e em seguida acesse a pagina pelo ip:
    
    ```docker
    http://172.31.212.213:80
    ```
    

1. **Criando e rodando um container interativo**
    - Inicie um container **Ubuntu** e interaja com o terminal dele.
    - 🔹 *Exemplo de aplicação:* Teste um script Bash que imprime logs do sistema ou instala pacotes de forma interativa.
    
    Baixe a imagem do ubuntu:
    
    ```docker
    docker pull ubuntu
    ```
    
    Execute o ubuntu de forma independente e interativa:
    
    ```docker
    docker run -di --name ubuntu ubuntu
    ```
    
    Entre no terminal do ubuntu:
    
    ```docker
    docker exec -ti ubuntu bash
    ```
    
    Atualize o ubuntu:
    
    ```bash
    apt update
    apt upgrade
    apt full-upgrade
    ```
    
    Instale o nano:
    
    ```bash
    apt install nano
    ```
    
    Crie o script:
    
    ```docker
    nano install_packages.sh
    ```
    
    Cole o script:
    
    ```bash
    #!/bin/bash
    if [ "$EUID" -ne 0 ]; then
        echo "Please run as root"
        exit 1
    fi
    
    echo "=== package installer ==="
    
    while true; do
        echo -e "\nType the name of the package to be installed (or type exit to quit): "
        read package_name
    
        if [ "$package_name" = "exit" ]; then
            echo "Exiting..."
            break
        fi
    
        #verifies if the package is already installed
        if dpkg -l | grep -qw "$package_name"; then
            echo "Package $package_name is already installed."
        else
            echo "Installing $package_name"
            apt update && apt install -y "$package_name"
    
            if dpkg -l | grep -qw "$package_name"; then
                echo "Package $package_name installed successfully."
            else
                echo "Failed to install package $package_name."
            fi
        fi
    done
    ```
    
    Adicione a permissão de execução para o script:
    
    ```bash
    chmod +x install_packages.sh
    ```
    
    Execute o script:
    
    ```bash
    ./install_packages.sh
    ```
    
2. **Listando e removendo containers**
    - Liste todos os containers em execução e parados, pare um container em execução e remova um container específico.
    - 🔹 *Exemplo de aplicação:* Gerenciar containers de testes criados para verificar configurações ou dependências.
    
    Listar containers:
    
    ```bash
    docker ps -a
    ```
    
    Pare um container em execução:
    
    ```bash
    docker stop [id ou nome do container]
    ```
    
    Remova um container:
    
    ```bash
    docker rm [id ou nome do container]
    ```
    
3. **Criando um Dockerfile para uma aplicação simples em Python**
    - Crie um `Dockerfile` para uma aplicação **Flask** que retorna uma mensagem ao acessar um endpoint.
    - 🔹 *Exemplo de aplicação:* Use a API de exemplo [Flask Restful API Starter](https://github.com/gothinkster/flask-realworld-example-app) para criar um endpoint de teste.
    
    Crie uma pasta para armazenar o arquivo `dockerfile`
    
    ```bash
    mkdir -p images/python
    cd images/python
    ```
    
    Crie o arquivo python:
    
    ```bash
    sudo nano app.py
    ```
    
    Cole o codigo:
    
    ```python
    from flask import Flask
    app = Flask(__name__)
    
    @app.route('/')
    def hello():
    	return "Hello World!"
    
    if __name__ == '__main__':
    	app.run(host='0.0.0.0', port=8000)
    ```
    
    Crie o arquivo dockerfile:
    
    ```bash
    sudo nano dockerfile
    ```
    
    Cole:
    
    ```bash
    # syntax=docker/dockerfile:1.4
    FROM --platform=$BUILDPLATFORM python:3.10-alpine AS builder
    
    WORKDIR /app
    
    RUN --mount=type=cache,target=/root/.cache/pip \
        pip3 install flask
    
    COPY . /app
    
    ENTRYPOINT ["python3"]
    CMD ["app.py"]
    
    FROM builder as dev-envs
    
    RUN <<EOF
    apk update
    apk add git
    EOF
    
    RUN <<EOF
    addgroup -S docker
    adduser -S --shell /bin/bash --ingroup docker vscode
    EOF
    # install Docker tools (cli, buildx, compose)
    COPY --from=gloursdocker/docker / /
    ```
    
    Crie a imagem:
    
    ```bash
    docker build -t appflask .
    ```
    
    Execute a imagem:
    
    ```bash
    docker run -ti --name app appflask
    ```
    

---

### 🟡 **Médio**

1. **Criando e utilizando volumes para persistência de dados**
    - Execute um container **MySQL** e configure um volume para armazenar os dados do banco de forma persistente.
    - 🔹 *Exemplo de aplicação:* Use o sistema de login e cadastro do [Laravel Breeze](https://github.com/laravel/breeze), que usa MySQL.
    
    ```bash
     docker pull mysql
     mkdir -p /data/mysql
     docker run -e MYSQL_ROOT_PASSWORD=0000 --name mysql -d -p 3306:3306 --volume=/data/mysql:/var/lib/mysql mysql
    ```
    
2. **Criando e rodando um container multi-stage**
    - Utilize um **multi-stage build** para otimizar uma aplicação **Go**, reduzindo o tamanho da imagem final.
    - 🔹 *Exemplo de aplicação:* Compile e rode a API do [Go Fiber Example](https://github.com/gofiber/recipes/tree/main/docker-multistage-build) dentro do container.
    
    No diretório do projeto, crie o arquivo GO:
    
    ```bash
    nano app.go
    ```
    
    ```go
    package main
    import ("fmt")
    func main(){
            fmt.Println("Qual o seu nome: ")
            var name string
            fmt.Scanln(&name)
            fmt.Printf("Oi, %s! Eu sou um app em GO!", name)
    }
    
    ```
    
    Crie o arquivo Dockerfile:
    
    ```bash
    nano dockerfile
    ```
    
    ```bash
    FROM golang as exec
    COPY app.go /go/src/app/
    ENV  GO111MODULE=auto
    WORKDIR /go/src/app/
    RUN go build -o app.go .
    FROM alpine
    WORKDIR /appexec
    COPY --from=exec /go/src/app /appexec
    RUN chmod -R 755 /appexec
    ENTRYPOINT ./app.go
    ```
    
    Crie a imagem:
    
    ```bash
    docker image build -t app-go:1.0 .
    ```
    
    Execute o container:
    
    ```bash
    docker run -ti --name meuappgo app-go:1.0
    ```
    
    ```bash
    # em /images/go:
    nano app.go
    
    package main
    import ("fmt")
    func main(){
            fmt.Println("Qual o seu nome: ")
            var name string
            fmt.Scanln(&name)
            fmt.Printf("Oi, %s! Eu sou um app em GO!", name)
    }
    
    nano dockerfile
    FROM golang as exec
    COPY app.go /go/src/app/
    ENV  GO111MODULE=auto
    WORKDIR /go/src/app/
    RUN go build -o app.go .
    FROM alpine
    WORKDIR /appexec
    COPY --from=exec /go/src/app /appexec
    RUN chmod -R 755 /appexec
    ENTRYPOINT ./app.go
    
    docker image build -t app-go:1.0 .
    docker run -ti --name meuappgo app-go:1.0
    ```
    
3. **Construindo uma rede Docker para comunicação entre containers**
    - Crie uma rede Docker personalizada e faça dois containers, um **Node.js** e um **MongoDB**, se comunicarem.
    - 🔹 *Exemplo de aplicação:* Utilize o projeto [MEAN Todos](https://github.com/luanphandinh/mean-todo) para criar um app de tarefas usando Node.js + MongoDB.
    
    Crie a rede a ser usada para comunicação:
    
    ```bash
    docker network create rede1
    ```
    
    No diretório da aplicação, iniciar o ambiente Node (E necessário ter o npm instalado → `apt install -y npm`)
    
    ```bash
    npm init
    ```
    
    Agora, modifique o arquivo package.json para instalar suas dependências e criar o script de inicialização:
    
    ```bash
    nano package.json
    ```
    
    ```json
      {
      "name": "appnodemongo",
      "version": "1.0.0",
      "description": "",
      "main": "app.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "start": "node app.js"
      },
      "dependencies": {
        "express": "^4.17.1",
        "mongoose": "^6.0.0",
        "nodemon": "^2.0.7"
      },
      "author": "",
      "license": "ISC"
    }
    ```
    
    Crie o arquivo `app.js` :
    
    ```bash
    nano app.js
    ```
    
    ```jsx
    const express = require('express');
    const mongoose = require('mongoose');
    
    const app = express();
    const port = 3000;
    
    // Conectar ao MongoDB
    mongoose.connect("mongodb://meu_mongo:27017/app_node")
      .then(() => console.log('Conectado ao MongoDB'))
      .catch(err => console.error('Erro ao conectar ao MongoDB:', err));
    
    // Rota de teste
    app.get('/', (req, res) => {
      res.send('Aplicação Node.js com MongoDB via Docker!');
    });
    
    app.listen(port, () => {
      console.log(`Servidor rodando na porta ${port}`);
    });
    
    ```
    
    <aside>
    ⚠️
    
    Detalhe: na string de conexão, `meu_mongo` representa o nome do seu container mongo que sera criado e `app_node` o nome do BD
    
    </aside>
    
    Crie o arquivo `Dockerfile`:
    
    ```bash
    nano dockerfile
    ```
    
    ```bash
    # Use uma imagem base oficial do Node (versão 14 em ambiente Alpine, que é leve)
    FROM node:14-alpine
    
    # Define o diretório de trabalho dentro do container
    WORKDIR /usr/src/app
    
    # Copia os arquivos de dependência para o container
    COPY package*.json ./
    
    # Instala as dependências definidas no package.json
    RUN npm install
    
    # Copia o restante dos arquivos do projeto para o container
    COPY . .
    
    # Expõe a porta que a aplicação irá utilizar (ex: 3000)
    EXPOSE 3000
    
    # Define o comando para iniciar a aplicação
    CMD ["npm", "start"]
    
    ```
    
    Construa a imagem Node:
    
    ```bash
    docker build -t minha_aplicacao_node .
    ```
    
    Crie e execute o container Mongo (a imagem sera baixada automaticamente):
    
    ```bash
    docker run -d --name meu_mongo --network rede1 mongo
    ```
    
    Execute o container Node indicando a porta a ser usada:
    
    ```bash
    docker run -d --name appNode -p 3000:3000 --network rede1 minha_aplicacao_node
    ```
    
    Teste a aplicação com:
    
    ```bash
    curl http://localhost:3000/
    ```
    
    Para saber se a conexão com o Mongo esta feita:
    
    ```bash
    docker logs appNode
    ```
    
4. **Criando um compose file para rodar uma aplicação com banco de dados**
    - Utilize **Docker Compose** para configurar uma aplicação **Django** com um banco de dados **PostgreSQL**.
    - 🔹 *Exemplo de aplicação:* Use o projeto [Django Polls App](https://github.com/databases-io/django-polls) para criar uma pesquisa de opinião integrada ao banco.
    
    Na pasta do projeto instale:
    
    ```bash
    sudo apt install python3.12-venv
    ```
    
    Depois:
    
    ```bash
    python3 -m venv venv
    source venv/bin/activate
    ```
    
    instale o DJANGO:
    
    ```bash
    
    pip install django
    ```
    
    Inicie o projeto:
    
    ```bash
    
    django-admin startproject meuprojeto
    ```
    
    No diretorio `meuprojeto`
    
    ```bash
    python [manage.py](http://manage.py/) startapp polls
    ```
    
    Crie o arquivo `settings.py`
    
    ```bash
    
    nano [settings.py](http://settings.py/)
    ```
    
    ```bash
    INSTALLED_APPS = [
    ...
    'polls',
    ]
    ```
    
    Saia do diretorio com `cd ..`
    
    Instale:
    
    ```bash
    
    pip install psycopg2-binary
    ```
    
    E crie novamente:
    
    ```bash
    
    nano [settings.py](http://settings.py/)
    ```
    
    ```bash
    DATABASES = {
    	'default': {
    	'ENGINE': 'django.db.backends.postgresql',
    	'NAME': 'mydb',
    	'USER': 'felipe',
    	'PASSWORD': '123',
    	'HOST': 'db',
    	'PORT': '5432',
    	}
    }
    ```
    
    Volte para a pasta do projeto: `cd meuprojeto`
    Crie o `docker-compose`
    
    ```bash
    
    nano docker-compose.yml
    ```
    
    ```bash
    version: '3.8'
    
    services:
      db:
        image: postgres:13
        volumes:
          - postgres_data:/var/lib/postgresql/data
        environment:
          POSTGRES_DB: mydb
          POSTGRES_USER: arthur
          POSTGRES_PASSWORD: 0000
        networks:
          - rede2
    
      web:
        build: .
        command: python manage.py runserver 0.0.0.0:8000
        volumes:
          - .:/app
        ports:
          - "8000:8000"
        depends_on:
          - db
        networks:
          - rede2
    volumes:
    	 postgres_data:
    
    networks:
      rede2:
    ```
    
    Crie o dockerfile:
    
    ```bash
    
    nano Dockerfile
    ```
    
    ```bash
    
    FROM python:3.9-slim
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt
    COPY . .
    EXPOSE 8000
    ```
    
    Crie o `requirements.txt`
    
    ```bash
    
    nano requirements.txt
    ```
    
    ```bash
    
    Django==3.2
    psycopg2-binary==2.9.3
    ```
    
    Crie o projeto:
    
    ```bash
    
    docker-compose up --build
    ```
    
    Abra um novo terminal e navegue até o diretório meuprojeto.
    
    ```bash
    docker-compose exec web python [manage.py](http://manage.py/) migrate
    ```
    
    Acessar a página do Django via navegador [http://localhost:8000](http://localhost:8000/).
    
    Teste o DB com:
    
    ```bash
    docker-compose exec db psql -U arthur -d mydb
    ```
    

---

### 🔴 **Difícil**

1. **Criando uma imagem personalizada com um servidor web e arquivos estáticos**
    - Construa uma imagem baseada no **Nginx** ou **Apache**, adicionando um site HTML/CSS estático.
    - 🔹 *Exemplo de aplicação:* Utilize a [landing page do Creative Tim](https://github.com/creativetimofficial/material-kit) para criar uma página moderna hospedada no container.

No diretório do projeto, crie o Dockerfile:

```bash
nano dockerfile
```

```bash
FROM alpine
RUN apk update && apk add git
RUN apk add --no-cache nginx
# arquivo de configuracao do nginx para garantir que a pagina certa seja exibida
COPY nginx.conf /etc/nginx/nginx.conf

RUN mkdir -p /usr/share/nginx/html

#clona o repo
RUN git clone https://github.com/creativetimofficial/material-kit.git /usr/share/nginx/html

#move o repo para a pasta de trabalho do nginx
RUN mv /usr/share/nginx/html/material-kit/* /usr/share/nginx/html && \
    mv /usr/share/nginx/html/material-kit/.* /usr/share/nginx/html || true

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Crie tambem o arquivo de configuração do nginx: `nginx.conf`

```bash
nano nginx.conf

```

```bash
worker_processes 1;

events {
  worker_connections 1024;
}

http {
  include       mime.types;
  default_type  application/octet-stream;

  sendfile        on;
  keepalive_timeout  65;

  server {
    listen 80;

    root /usr/share/nginx/html;  # Diretório correto onde o index.html está
    index index.html index.htm;

    location / {
      try_files $uri $uri/ /index.html;
    }

    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
  }
```

Construa a imagem:

```bash
docker build -t nginx-alpine .
```

Crie o container:

```bash
docker run -d --name serverNginx -p 8080:80 nginx-alpine
```

No terminal do Windows (nao no WSL), procure o IP do WLS com:

```bash
ipconfig
```

Copie o IPv4 com o titulo: `Adaptador Ethernet vEthernet (WSL (Hyper-V firewall)):`

Acesse a pagina pelo navegador:

[http://ip-da-maquina:8080/](http://172.31.208.1:8080/)
