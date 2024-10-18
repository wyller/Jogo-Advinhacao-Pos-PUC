# Jogo de Adivinhação com Flask e React

Este é um jogo de adivinhação, onde você tenta descobrir uma senha gerada aleatoriamente. Ele foi feito usando Flask no backend (parte do servidor) e React no frontend (parte visual). O jogo te dá dicas sobre quantas letras da sua tentativa estão corretas e se estão nas posições certas.

## O que o jogo faz

- Você pode criar um novo jogo com uma senha secreta.
- Tente adivinhar a senha e receba dicas sobre quais letras estão certas e se estão no lugar certo.
- As senhas são guardadas de forma segura usando base64.
- Se você errar, o jogo te dá dicas para tentar de novo.

## O que você precisa para rodar o jogo

- Docker 20.10.0 ou mais recente
- Docker Compose 3.9 ou mais recente

## Como rodar o jogo

1. Primeiro, clone o repositório (código do jogo) para o seu computador:

   ```bash
   git clone https://github.com/wyller/Jogo-Advinhacao-Pos-PUC.git
   cd Jogo-Advinhacao-Pos-PUC
   ```

2. Depois, rode o jogo usando Docker Compose:

   ```bash
   docker compose up -d
   ```

3. Agora, abra o navegador e acesse o jogo em: [http://localhost:80](http://localhost:80) 🎮

## Como jogar

### 1. Criar um novo jogo

- Acesse: [http://localhost:80/maker](http://localhost:80/maker)
- Digite uma senha secreta.
- Envie e guarde o `game-id` que será gerado (você vai precisar dele depois).

### 2. Adivinhar a senha

- Acesse: [http://localhost:80/breaker](http://localhost:80/breaker)
- Insira o `game-id` que você salvou.
- Tente adivinhar a senha!

## Como o jogo funciona com Docker

### Estrutura do Projeto

O jogo está dividido em três partes principais, chamadas de "serviços", que são gerenciadas pelo Docker Compose:

#### Serviços:

```yaml
services:
  db:
  backend:
  frontend:
```

- **db**: É o banco de dados PostgreSQL, onde as informações do jogo são guardadas. Ele tem um sistema que verifica se está funcionando corretamente e usa um volume para salvar os dados, mesmo que o container seja reiniciado.

```yml
db:
  image: postgres:14.13
  restart: always
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U postgres -d postgres"]
    interval: 3s
    retries: 5
    start_period: 30s
  secrets:
    - db-password
  volumes:
    - db-data:/var/lib/postgresql/data
  networks:
    - backnet
  environment:
    - POSTGRES_PASSWORD=/run/secrets/db-password
  expose:
    - 5432
```

- **backend**: É o servidor Flask, que cuida da lógica do jogo e se comunica com o banco de dados. Ele só começa a funcionar depois que o banco de dados estiver pronto.

```yml
backend:
  build:
    context: backend
  restart: always
  secrets:
    - db-password
  environment:
    - FLASK_DB_PASSWORD=/run/secrets/db-password
    - FLASK_DB_HOST=db
  expose:
    - 5000
  networks:
    - backnet
    - frontnet
  depends_on:
    db:
      condition: service_healthy
```

- **frontend**: É o servidor Nginx, que mostra a interface do jogo (React) e faz a conexão entre o frontend e o backend.

```yml
frontend:
  build:
    context: frontend
  restart: always
  ports:
    - 80:80
  depends_on:
    - backend
  networks:
    - frontnet
```

#### Volumes

O volume `db-data` é usado para garantir que os dados do banco de dados não sejam perdidos, mesmo que o container seja reiniciado.

```yml
volumes:
  db-data:
```

#### Redes

O projeto usa duas redes:

- **backnet**: Conecta o banco de dados ao backend.
- **frontnet**: Conecta o frontend ao backend.

```yml
networks:
  backnet:
  frontnet:
```

#### Balanceamento de Carga

O Nginx, além de servir o frontend, também age como um "mensageiro", encaminhando as requisições do frontend para o backend.

```nginx
upstream backend {
  server backend:5000;
}

server {
  listen 80;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;
  }

  location /api {
    proxy_pass http://backend/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

Agora é só seguir os passos e começar a jogar! 😄