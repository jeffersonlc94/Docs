# Manual de Troca de Domínio Whazing

## 1. Fazer um backup  SnapShot da sua VPS

## 2. Alterando para root
```bash
sudo su root
```

## 3. Parar todos os processos do PM2
```bash
pm2 stop all
```

## 4. Editar o arquivo `.env` do backend
```bash
nano /whazing/backend/.env
```
edite seu dominio backend:
```env
URL_API='https://backend.seusite.com.br'
ID_DO_APP_DO_FACEBOOK='23156312477653241'
```

## 5. Editar o arquivo `.env` do frontend
```bash
nano /whazing/frontend/.env
```
edite seu dominio frontend e backend:
```
# URL do backend para construção dos hooks
BACKEND_URL=https://backend.seusite.com.br

# URL do frontend para liberação do CORS
FRONTEND_URL=https://frontend.seusite.com.br
```

## 6. Editar configuração do Nginx para o backend
```bash
sudo nano /etc/nginx/sites-available/whazing-backend
```
Altere as linhas `server_name` e `server`   NÃO MODIFIQUE OS SSL pois serao recriados:
```nginx
server {
    server_name backend.seusite.com.br;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # Gerenciado pelo Certbot
    ssl_certificate /etc/letsencrypt/live/backend.seusite.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/backend.seusite.com.br/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    if ($host = backend.seusite.com.br) {
        return 301 https://$host$request_uri;
    }
}

server {
    server_name backend.seusite.com.br;
    listen 80;
    return 404;
}
```

## 7. Editar configuração do Nginx para o frontend
```bash
sudo nano /etc/nginx/sites-available/whazing-frontend
```
Altere  as linhas `server_name` e `server` NÃO MODIFIQUE OS SSL pois serao recriados:
```nginx
server {
    server_name frontend.seusite.com.br;

    location / {
        proxy_pass http://127.0.0.1:3333;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # Gerenciado pelo Certbot
    ssl_certificate /etc/letsencrypt/live/frontend.seusite.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/frontend.seusite.com.br/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    if ($host = frontend.seusite.com.br) {
        return 301 https://$host$request_uri;
    }
}

server {
    server_name frontend.seusite.com.br;
    listen 80;
    return 404;
}
```
## 8. Gerar certificado
```bash
sudo certbot --nginx
```
## 9. Reinicia  VPS
```bash
reboot
```
## 10. Inicia PM
```bash
pm2 flush all
pm2 start all
```
