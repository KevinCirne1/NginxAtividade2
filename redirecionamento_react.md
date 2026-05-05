# Tutorial 1: Configurando Rotas ReactJS no Nginx com Docker

Este tutorial detalha como configurar o Nginx para servir uma aplicação React (utilizando React Router) corretamente, garantindo que o redirecionamento de rotas funcione sem erros 404 ao atualizar a página.

## 🎯 O Problema
Aplicações React são **Single Page Applications (SPAs)**. Isso significa que o roteamento é feito no navegador pelo JavaScript. Quando você acessa uma rota como `/dashboard` e atualiza a página (F5), o Nginx tenta encontrar um arquivo físico chamado `dashboard` na pasta do servidor. Como esse arquivo não existe, o servidor retorna o erro **404 Not Found**.

## 💡 A Solução
A solução consiste em configurar o Nginx para que, caso ele não encontre o arquivo solicitado, ele redirecione a requisição para o `index.html`. A partir daí, o React Router assume o controle da URL e renderiza o componente correto.

## 🛠️ Passo a Passo com Docker

### Passo 1: O web.Dockerfile (Multi-stage)
Para garantir eficiência, utilizamos o build em dois estágios:

```dockerfile
# Estágio 1: Build da aplicação
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Estágio 2: Servidor Nginx
FROM nginx:stable-alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
Passo 2: O nginx.conf (A Configuração Chave)
Crie um arquivo nginx.conf na raiz do projeto com a diretiva try_files:

Nginx
server {
    listen 80;
    
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        # Tenta servir o arquivo, depois o diretório. 
        # Se nada for encontrado, envia para o index.html
        try_files $uri $uri/ /index.html;
    }
}
Passo 3: Execução
Suba o ambiente com o Docker Compose:

docker-compose up -d --build
Passo 4: Verificação
Acesse http://localhost:8080/qualquer-rota.

Pressione F5.

Se a página recarregar sem erro 404, a configuração foi um sucesso!