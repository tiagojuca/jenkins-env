# qjenkins
  
Local da senha padrao:  
`/var/lib/docker/volumes/qjenkins-data/_data/secrets/initialAdminPassword`  
  
## Procedimento manual:  
Compilar imagem:  
`docker build -t qjenkins:2.550-jdk21 .`  
  
Criar rede compartilhada:  
`docker network create jenkins`  
  
Executar container:  
`docker run --name qjenkins --restart=on-failure --detach \`  
`  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \`  
`  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \`  
`  --publish 8080:8080 --publish 50000:50000 \`  
`  --volume jenkins-data:/var/jenkins_home \`  
`  --volume jenkins-docker-certs:/certs/client:ro \`  
`  qjenkins:2.550-jdk21`  
  
Diferente da abordagem subsequente via Docker Compose, a execucao manual persiste os
artefatos gerados (containers, imagens, volumes, redes etc), sendo necessario remove-los
manualmente para que a infraestrutura seja realmente efemera. Isso pode ser desejado,
mas em geral persistimos apenas os volumes e o resto eh sempre recriado cada vez que sobe  
  
Comandos uteis:  
`docker ps`  
`docker ps -a` ou `docker container ls -a`  
`docker ps -q`  
`docker ps -a --filter "status=exited"`  
`docker logs NAME`  
`docker run IMAGE_NAME:TAG`  
`docker start NAME`  
`docker stop NAME`  
`docker kill NAME`  
`docker restart NAME`  
`docker rm NAME`  
`docker rm -f NAME`  
`docker container prune -f`  
`docker image ls` ou `docker images`  
`docker image rm ID` ou `docker rmi ID`  
`docker image prune -f`  
`docker image prune -a -f` (CUIDADO!)  
`docker build -t IMAGE_NAME:TAG .`  
`docker network ls`  
`docker network create NAME`  
`docker network rm NAME`  
`docker volume ls`  
`docker volume rm VOLUME_NAME`  
`docker volume prune -f`  
`docker volume prune -a -f` (CUIDADO!)  
  
## Docker Compose
Docker Compose eh mais automatizado, utilizado para orquestracao de containers.  
Por padrao, os volumes sao persistidos, eh claro, mas podem ser apagados com a flag `-v`.  
Idem para as imagens, que podem ser apagadas com a flag `--rmi`, podendo passar `local` ou `all`.  
  
Cheatsheet Docker Compose:  
`docker compose ps`  
`docker compose up -d` (eh bom passar -d para o dettach)  
`docker compose stop`  
`docker compose down`  
`docker compose restart`  
  
Como estamos desenvolvendo, ao subir o compose, eh interessante passar a flag
`--build` para forcar a reconstrucao das imagens de servicos alterados:  
`docker compose up -d --build`  
  
Eh possivel apenas parar os servicos, mantendo os containers, com o comando `stop`,
mas a ideia toda de docker eh gerar ambientes virtuais efemeros, entao o mais comum
eh utilizar `down`, que remove os containers e redes do ambiente  
  
Se quiser apagar tambem os volumes,  
`docker compose down -v`  
  
Se quiser apagar tambem as imagens,  
`docker compose down -v --rmi local`  
ou  
`docker compose down -v --rmi all`  
  
Em geral nao faz muito sentido apagar as imagens baixadas do registry, entao se for
por questoes de cache, eh suficiente limpar as imagens construidas localmente (via Dockerfile).
Na verdade, nem isso costuma ser necessario, ja que, como indicado acima, podemos subir
a orquestracao com a flag `--build`, o que garante que as imagens sejam reconstruidas se
necessario, e eh ate mais eficiente, pois reconstroi APENAS se for necessario (Dockerfile mudou,
dependencias de build mudaram, imagens nao existem localmente etc), enquanto o primeiro
garante o rebuild total, mesmo se nao for necessario.  
  
A maior parte disso so eh relevante durante o desenvolvimento. Em producao, o que importa eh:  
`docker compose up -d`  
`docker compose down`  

## Links relacionados para leitura
- https://www.jenkins.io/doc/book/installing/docker/
- https://www.reviewboard.org/docs/manual/latest/admin/installation/docker/
- https://www.reviewboard.org/docs/reviewbot/latest/installation/docker/
