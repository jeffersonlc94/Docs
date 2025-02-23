# Manual de Troca de Domínio Whazing
Antes de fazer o procedimento faça os apontamentos dos domínios para sua vps.
## 1. Fazer um backup  SnapShot da sua VPS

## 2. Alterando para deploy
```bash
sudo su deploy
```

## 3. Parar todos os processos do PM2
```bash
pm2 stop all
```

## 4. Parar serviço nginx
```bash
sudo service nginx stop
```

## 5. Editar o arquivo `.env` do backend
```bash
nano /home/deploy/whazing/backend/.env
```
edite seu dominio backend:
```env
URL_API='https://backend.seusite.com.br'
ID_DO_APP_DO_FACEBOOK='23156312477653241'
```

## 6. Editar o arquivo `.env` do frontend
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

## 7. Editar configuração do Nginx para o backend
```bash
sudo nano /etc/nginx/sites-available/whazing-backend
```
Altere as linhas `server_name` e `server`   NÃO MODIFIQUE OS SSL pois serao recriados:
`nginx
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
`

## 8. Editar configuração do Nginx para o frontend
```bash
sudo nano /etc/nginx/sites-available/whazing-frontend
```
Altere  as linhas `server_name` e `server` NÃO MODIFIQUE OS SSL pois serao recriados:
`nginx
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
`
## 9. Gerar certificado
selecione o dominio (faça isso para os dois domínios, backend e frontend)
```bash
sudo certbot --nginx
```
## 10. Reinicia  VPS
```bash
reboot
```
## 11. Inicia PM2
```bash
pm2 flush all
pm2 start all
```
## 12. Se o ngix nao iniciar no reboot
```bash
sudo service nginx status
sudo service nginx start
```
## 13. Atualizar o sistema
Faça a atualização do sistema.

depois limpe os cookies do navegador.

materiais de apoio

https://github.com/cleitonme/Whazing-SaaS/blob/main/docs/INSTALL_VPS_UBUNTU_20_22.md

https://github.com/cleitonme/Whazing-SaaS.instalador



