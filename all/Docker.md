Терминология

- _Images_ - The blueprints of our application which form the basis of containers. In the demo above, we used the `docker pull` command to download the **busybox** image.
- _Containers_ - Created from Docker images and run the actual application. We create a container using `docker run` which we did using the busybox image that we downloaded. A list of running containers can be seen using the `docker ps` command.
- _Docker Daemon_ - The background service running on the host that manages building, running and distributing Docker containers. The daemon is the process that runs in the operating system which clients talk to.
- _Docker Client_ - The command line tool that allows the user to interact with the daemon. More generally, there can be other forms of clients too - such as [Kitematic](https://kitematic.com/) which provide a GUI to the users.
- _Docker Hub_ - A [registry](https://hub.docker.com/explore/) of Docker images. You can think of the registry as a directory of all available Docker images. If required, one can host their own Docker registries and can use them for pulling images.

всё под sudo, если не сконфигурирована группа для докера,
которая пустая по дефолту

$ docker pull name_of_image
$ docker images
$ docker run name_of_image
$ docker ps -a  == $ docker container ls -a 
$ docker run -it busybox sh (run -it открывает терминал внутри контейнера)
exit (выход из терминала внутри контейнера) 

$ docker rm 
$ docker rm $(docker ps -a -q -f status=exited)
(-q возвращает только id, -f фильтр по условию)
$ docker container prune (тот же результат)
$ docker rmi name_of_image (буквально полное удаление) 
$ docker run --rm -it prakhar1989/static-site
(--rm удаляет image, если он есть локально)

$ docker run -d -P --name static-site prakhar1989/static-site
In the above command, `-d` will detach our terminal, `-P` will publish all exposed ports to random ports and finally `--name` corresponds to a name we want to give. Now we can see the ports by running the `docker port [CONTAINER]` command
$ docker port static-site
$ docker run -p 8888:80 prakhar1989/static-site
(specified port for container: "You can also specify a custom port to which the client will forward connections to the container.")

$ docker stop name_of_the_container
$ docker start name_of_the_container

$ docker build -t yourusernameondockerhub/nameofthenewimage location_of_dockerfile

$ docker login
$ docker push local_image_name

$ docker search name_of_image

$ docker exec -it name_of_the_container /bin/sh(запуск терминала в работающем контейнере. exec вообще позволяет просто запустить команду внутри контейнера, а дальше по накатанной)
$ docker cp file.txt container-name:/path/**to**/**copy**/file.txt(скопировать файл с хост машины в работающий контейнер)
$ docker exec --user nikita -it 8aaf92c49202 /bin/sh (запуск контейнера под определенным пользователем
*было полезно для запуска развертывания тестовой базы для постгресс*)

$ docker info
$ docker container logs name_of_the_container

networking in docker
$ docker network ls
$ docker network inspect name_of_the_network
$ docker network create name_of_the_net 
$ docker run -d --name es --net net_name -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" image_name

docker compose
$ docker-compose up (inside the directory with .yml file) 
$ docker-compose down -v (To destroy the cluster and the data volumes, just type)
$ docker-compose run name_of_the_app bash


https://habr.com/ru/articles/578744/