# Bagisto com Docker (Nginx + PHP-FPM + MariaDB)

Este repositório entrega um ambiente **Docker totalmente funcional** para rodar o **Bagisto** (Laravel e-commerce) usando:

* Nginx
* PHP-FPM
* MariaDB
* PHPMyAdmin

O objetivo deste guia é permitir que qualquer pessoa clone o repositório e chegue a um **Bagisto 100% funcional**, evitando erros comuns de **permissão**, **.env**, **404 no Nginx** e **conexão com banco**.

---

## 1. Pré-requisitos

* Docker
* Docker Compose (plugin `docker compose`)
* Git

Verifique:

```bash
docker --version
docker compose version
git --version
```

---

## 2. Clonar o repositório

```bash
git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
cd bagisto-docker
```

Estrutura esperada:

```
bagisto-docker/
├── docker-compose.yml
├── Dockerfile
├── .env
├── workspace/
│   └── bagisto/
└── .configs/
    └── nginx/
        └── nginx.conf
```

---

## 3. Configurar o arquivo `.env` (Docker Compose)

O arquivo `.env` **deve ficar no mesmo diretório do `docker-compose.yml`**.

Exemplo mínimo:

```env
DB_ROOT_PASSWORD=root
DB_DATABASE=bagisto
DB_USERNAME=bagisto_user
DB_PASSWORD=bagisto_pass
```

> ⚠️ Se esse arquivo não existir, o Docker exibirá warnings como:
> `The "DB_PASSWORD" variable is not set`

---

## 4. Subir os containers

```bash
docker compose build --no-cache
docker compose up -d
```

Verifique se tudo está rodando:

```bash
docker compose ps
```

Containers esperados:

* bagisto-php
* bagisto-nginx
* bagisto-mysql
* bagisto-phpmyadmin

---

## 5. Instalar o Bagisto no container PHP

Entre no container PHP:

```bash
docker exec -it bagisto-php sh
```

Acesse o diretório do Bagisto:

```bash
cd /var/www/html/bagisto
```

### 5.1 Criar o `.env` do Laravel

Se não existir:

```bash
cp .env.example .env
```

Gerar a key:

```bash
php artisan key:generate
```

### 5.2 Ajustar banco no `.env`

Confirme que os dados batem com o Docker:

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=bagisto
DB_USERNAME=bagisto_user
DB_PASSWORD=bagisto_pass
```

---

## 6. Corrigir permissões (PASSO OBRIGATÓRIO)

Este passo evita o erro:

```
Failed to open stream: Permission denied
```

Execute **dentro do container PHP**:

```bash
chown -R www-data:www-data storage bootstrap/cache
chmod -R 775 storage bootstrap/cache
```

Limpar caches:

```bash
php artisan optimize:clear
php artisan view:clear
php artisan config:clear
```

---

## 7. Rodar o instalador do Bagisto

Ainda dentro do container:

```bash
php artisan bagisto:install
```

Responda:

* Application name: Bagisto
* Application URL: [http://localhost](http://localhost)
* Timezone: America/Fortaleza
* Locale: Brazilian Portuguese
* Currency: Brazilian Real
* Database host: mysql
* Database port: 3306
* Database name: bagisto
* Database user: bagisto_user
* Database password: bagisto_pass

Ao final, o banco será populado com sucesso.

---

## 8. Configuração correta do Nginx

Arquivo:

```
.configs/nginx/nginx.conf
```

Conteúdo:

```nginx
server {
    listen 80;
    server_name localhost;

    root /var/www/html/bagisto/public;
    index index.php index.html;

    charset utf-8;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass bagisto-php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Reinicie o Nginx:

```bash
docker compose restart nginx
```

---

## 9. Acessos

* Loja: [http://localhost](http://localhost)
* Admin: [http://localhost/admin](http://localhost/admin)
* PHPMyAdmin: [http://localhost:8080](http://localhost:8080)

Credenciais padrão do Bagisto:

* Usuário: [admin@example.com](mailto:admin@example.com)
* Senha: admin123

---

## 10. Problemas comuns

### 404 no Nginx

* Root errado (deve ser `bagisto/public`)
* Nginx não reiniciado

### Permission denied

* Não rodou `chown` e `chmod`

### Banco não conecta

* `DB_HOST` deve ser **mysql** (nome do serviço)

---

## 11. Status final

✔ Docker configurado corretamente
✔ Bagisto instalado
✔ Nginx funcional
✔ Permissões corrigidas
✔ Pronto para desenvolvimento ou deploy

---

Se algo falhar, revise os passos **6 (permissões)** e **8 (Nginx)** — são os pontos mais críticos.
