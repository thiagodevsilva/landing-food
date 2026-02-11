# Publicar a LP na VPS (thiagosilva.dev.br)

## 1. Subir o projeto no GitHub

Execute no seu computador, dentro da pasta do projeto:

```bash
cd /home/thiagows72/landing-food

# Inicializar repositório (se ainda não tiver)
git init

# Adicionar todos os arquivos
git add .

# Primeiro commit
git commit -m "Landing page restaurantes - conversão e WhatsApp"

# Criar repositório no GitHub: https://github.com/new
# Nome sugerido: landing-food (ou outro)
# NÃO marque "Add README" se já tiver arquivos locais

# Adicionar o remote (troque SEU_USUARIO pelo seu usuário do GitHub)
git remote add origin https://github.com/SEU_USUARIO/landing-food.git

# Enviar para o GitHub (branch main)
git branch -M main
git push -u origin main
```

Se o GitHub pedir autenticação, use um **Personal Access Token** (Settings → Developer settings → Personal access tokens) no lugar da senha.

---

## 2. Publicar na VPS

### Opção A: LP na raiz do domínio (thiagosilva.dev.br = esta LP)

Na VPS:

```bash
# Conectar na VPS (troque pelo seu usuário e IP/host)
ssh usuario@thiagosilva.dev.br

# Ir até a pasta do site (caminho comum com nginx)
cd /var/www/html
# OU, se usar usuário e pasta em home:
# cd ~/thiagosilva.dev.br

# Clonar o repositório (troque SEU_USUARIO pelo seu usuário do GitHub)
sudo git clone https://github.com/SEU_USUARIO/landing-food.git .

# Se a pasta já existir e tiver outros arquivos, clone em subpasta e depois mova:
# sudo git clone https://github.com/SEU_USUARIO/landing-food.git /var/www/landing-food
# sudo cp -r /var/www/landing-food/* /var/www/html/
```

Configuração **nginx** para servir a LP na raiz:

```nginx
server {
    listen 80;
    server_name thiagosilva.dev.br www.thiagosilva.dev.br;
    root /var/www/html;
    index index.html;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Recarregar nginx:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

### Opção B: LP em subdomínio (ex: lp.thiagosilva.dev.br ou restaurante.thiagosilva.dev.br)

Na VPS:

```bash
ssh usuario@thiagosilva.dev.br
sudo mkdir -p /var/www/landing-food
cd /var/www/landing-food
sudo git clone https://github.com/SEU_USUARIO/landing-food.git .
```

Criar config do nginx para o subdomínio:

```bash
sudo nano /etc/nginx/sites-available/landing-food
```

Conteúdo (troque `lp` pelo subdomínio que quiser, ex: `restaurante`):

```nginx
server {
    listen 80;
    server_name lp.thiagosilva.dev.br;
    root /var/www/landing-food;
    index index.html;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Ativar e recarregar:

```bash
sudo ln -s /etc/nginx/sites-available/landing-food /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

Criar o registro **A** (ou CNAME) do subdomínio no painel do domínio apontando para o IP da VPS.

---

### Opção C: LP em uma pasta (thiagosilva.dev.br/landing-food)

Colocar os arquivos em uma subpasta do site atual:

```bash
ssh usuario@thiagosilva.dev.br
cd /var/www/html
sudo mkdir -p landing-food
cd landing-food
sudo git clone https://github.com/SEU_USUARIO/landing-food.git .
sudo mv index.html ./
# Remover pasta .git se não quiser na pasta pública (opcional)
# sudo rm -rf .git
```

Acesso: **https://thiagosilva.dev.br/landing-food/**

---

## 3. Atualizar o site depois de mudanças

Sempre que fizer alterações e der push no GitHub:

```bash
ssh usuario@thiagosilva.dev.br
cd /var/www/html   # ou /var/www/landing-food na opção B
sudo git pull origin main
```

---

## 4. HTTPS (recomendado)

Se ainda não tiver certificado no domínio:

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d thiagosilva.dev.br -d www.thiagosilva.dev.br
```

Para subdomínio: `sudo certbot --nginx -d lp.thiagosilva.dev.br`
