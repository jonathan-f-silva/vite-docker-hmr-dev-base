# Steps from scratch

## UI (vite app)
1. Create a empty folder
   
2. (optional) Create a git repo
   
3. Create a new vite app on a folder (named `ui` on this example)
   
4.  Add this config on `vite.config.js` on UI folder for exposing ports and set HMR port ([reference](https://github.com/vitejs/vite/issues/4116#issuecomment-983261927))
```ts
server: {
  host: true,
  hmr: {
    port: 3001,
  },
```

5. Add a `Dockerfile` ([reference](https://www.docker.com/blog/speed-up-your-development-flow-with-these-dockerfile-best-practices/))
```dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . ./
CMD npm run dev
```

6. Add a `.dockerignore`
```dockerignore
node_modules
dist
```
- On this point, you can build the UI docker image and run it binding the ports and it works (on the UI folder):
```sh
docker build . -t hmr-vite-ui
docker run --rm -it -v $(pwd)/src:/app/src -p 3000:3000 -p 3001:3001 hmr-vite-ui
```


## Reverse-proxy (NGINX) ([reference](https://github.com/okteto/movies-with-compose))
1. Make a new folder on root for the reverse-proxy NGINX server (named `reverse-proxy` on this example)
   
2. Create a file `default.conf` on reverse-proxy folder for overwriting the default NGINX config. This sets:
  - The upstream ports from the UI container for the UI and HMR
  - The NGINX main proxy for listen on port 80, redirecting all of it to UI container
  - The HMR proxy server ([reference](https://www.tutorialspoint.com/how-to-configure-nginx-as-reverse-proxy-for-websocket))
```nginx
upstream ui {
  server ui:3000;
}
upstream hmr {
  server ui:3001;
}
server {
  listen 80;
  location / {
      proxy_pass http://ui;
  }
}
server {
  listen 3001;
  location / {
      proxy_pass http://hmr;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
  }
}
```

3. Create a `Dockerfile` on the reverse-proxy folder:
```dockerfile
FROM nginx
COPY default.conf /etc/nginx/conf.d/default.conf
```

## Docker Compose

1. Create a file `docker-compose.yml` on root folder
- This brings together our two containers
- Expose ports 80 (main) e 3001 (HMR) for the host
- And set the volume for watching changes
```yml
services:
  reverse-proxy:
    build: reverse-proxy
    ports:
      - 80:80
      - 3001:3001
  ui:
    build: ui
    ports:
      - 3000
      - 3101
    volumes:
      - ./ui/src:/app/src    
```

2. Build and start it with (on root folder):
```sh
docker compose up
```

3. You can see the app running in port 80 (http://localhost:80) and can edit for the sweet HMR!
