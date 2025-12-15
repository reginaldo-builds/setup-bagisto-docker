# Bagisto Docker – Setup Manual (Local e Base para Produção)

Este repositório fornece um **ambiente Docker simples, estável e controlado manualmente** para rodar o **Bagisto v2.3.6+**, sem scripts automáticos frágeis.

O foco é:

* Ambiente **local funcional**
* Configuração **próxima de produção**
* Total controle sobre instalação e debug
* Compatibilidade futura com deploy (ex: Render)

---

## 📦 Serviços incluídos

Este setup utiliza apenas os serviços **necessários**:

* PHP 8.3 (PHP-FPM)
* Nginx
* MariaDB 10.6+
* phpMyAdmin (somente local)

> Serviços como Redis, Elasticsearch, Kibana e Mailpit **não foram incluídos** para manter o ambiente simples e previsível. Eles podem ser adicionados posteriormente se necessário.

---

## 🧩 Versão suportada do Bagisto

* **Bagisto:** v2.3.6 ou superior
* **PHP:** 8.3
* **Banco:** MariaDB 10.6+

---

## 📋 Requisitos do sistema

* Docker (última versão)
* Docker Compose v2 (`docker compose`)
* Linux, macOS ou Windows (com WSL2)

---

## 📁 Estrutura do projeto

```text
.
├── docker-compose.yml
├── Dockerfile
├── workspace/           # Código do Bagisto (não versionar)
├── .configs/
│   ├── nginx/
│   │   └── nginx.conf
│   └── .env
└── README.md
```

---

## ⚙️ Configuração de ambiente (.env)

Arquivo localizado em `.configs/.env`:

```env
APP_NAME=Bagisto
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost
APP_ADMIN_URL=admin
APP_TIMEZONE=America/Fortaleza

APP_LOCALE=pt_BR
APP_FALLBACK_LOCALE=pt_BR
APP_FAKER_LOCALE=pt_BR

APP_CURRENCY=BRL

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=bagisto
DB_USERNAME=bagisto_user
DB_PASSWORD=senha_segura_aqui
DB_ROOT_PASSWORD=senha_root_segura

SESSION_DRIVER=file
CACHE_STORE=file
QUEUE_CONNECTION=sync
FILESYSTEM_DISK=local
```

⚠️ **Nunca versionar esse arquivo com senhas reais**.

---

## 🐳 docker-compose.yml (resumo)

* PHP-FPM + Nginx separados
* MariaDB com volume persistente
* phpMyAdmin apenas para desenvolvimento local

> O arquivo completo deve ser revisado conforme seu ambiente.

---

## 🚀 Subida dos containers

```bash
docker compose up -d --build
```

Verifique se os containers estão rodando:

```bash
docker ps
```

---

## 📥 Instalação MANUAL do Bagisto (recomendado)

### 1️⃣ Acessar o container PHP

```bash
docker exec -it bagisto-php bash
```

---

### 2️⃣ Instalar o Bagisto

Dentro do container:

```bash
composer create-project bagisto/bagisto
cd bagisto
```

---

### 3️⃣ Copiar o arquivo `.env`

No host:

```bash
cp .configs/.env workspace/bagisto/.env
```

---

### 4️⃣ Gerar a chave da aplicação

```bash
php artisan key:generate
```

---

### 5️⃣ Ajustar permissões (obrigatório)

```bash
chmod -R 775 storage bootstrap/cache
```

---

### 6️⃣ Instalar o Bagisto

```bash
php artisan bagisto:install --skip-env-check
```

---

### 7️⃣ Criar link de storage

```bash
php artisan storage:link
```

---

## 🌐 Acesso ao sistema

* Loja:

  ```
  http://localhost
  ```

* Admin:

  ```
  http://localhost/admin
  ```

Credenciais padrão:

```text
Email: admin@example.com
Senha: admin123
```

---

## 🛠 phpMyAdmin (local)

```text
http://localhost:8080
```

* Host: mysql
* Usuário: bagisto_user
* Senha: definida no `.env`

---

## ❌ O que NÃO é usado neste setup

* Script `setup.sh`
* Git clone automático dentro do container
* Composer rodando em runtime automático
* Criação manual de banco via script

Essas abordagens foram removidas para garantir:

* previsibilidade
* facilidade de debug
* compatibilidade com produção

---

## 🧭 Próximos passos recomendados

* Criar `docker-compose.prod.yml`
* Ajustar Nginx (gzip, cache, headers)
* Build de assets para produção
* Deploy no Render (plano gratuito)

---

## ✅ Conclusão

Este setup oferece:

✔ Controle total ✔ Simplicidade ✔ Compatibilidade futura com produção ✔ Base sólida para escalar

Para dúvidas ou extensões (Redis, Elasticsearch, Render), continue a configuração conforme a necessidade do projeto.
