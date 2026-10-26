## Lab 1 — Instalacja Dockera

Polecenia wykonuj jako root, w podanej kolejności. System docelowy: CentOS Stream 9 lub 10.
```bash
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine \
                  podman \
                  runc
```


```bash
dnf install -y dnf-plugins-core
```

```bash
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

```bash
yum repolist
```

```bash
yum search docker
```

```bash
yum info docker-ce
```

```bash
yum install -y docker-ce
```
```bash
systemctl enable --now docker
usermod -a -G docker student
docker run hello-world

```


## Lab 2 — Weryfikacja instalacji i uruchomienie Dockera

Wykonuj jako root. Widok `systemctl status` i `systemctl cat` zamknij klawiszem `q`.

Pierwsze `docker info` może zgłosić brak połączenia z demonem — przed uruchomieniem usługi Docker jest to oczekiwane. Po `systemctl enable --now docker` kolejne sprawdzenie powinno się powieść.

```bash
docker --version
```

```bash
docker info
```

```bash
systemctl enable --now docker
```

```bash
docker version
```

```bash
systemctl status docker
```

```bash
docker info
```

```bash
systemctl cat docker
```

```bash
ls /var/lib/docker
```

## Lab 3 — Użytkownik admin i uprawnienia do Dockera

Zacznij jako root. `su - admin` przełącza na użytkownika admin, a `exit` wraca do roota. Po ostatnim `su - admin` kolejne laboratoria wykonuj jako admin.

Pierwsze `docker version` jako admin powinno zgłosić brak uprawnień do demona. Po dodaniu użytkownika do grup i ponownym `su - admin` polecenie powinno wyświetlić także dane serwera.

```bash
ls -l /var/run/docker.sock
```

```bash
useradd admin
```

```bash
passwd admin
```

```bash
su - admin
```

```bash
docker version
```

```bash
exit
```

```bash
usermod -aG docker,wheel admin
```

```bash
su - admin
```

```bash
docker version
```

## Lab 4 — Pierwszy kontener

```bash
docker run hello-world
```

## Lab 5 — Lista kontenerów i obrazów

```bash
docker ps
```

```bash
docker ps -a
```

```bash
docker images
```

## Lab 6 — Apache i procesy Dockera

```bash
docker container run -d httpd
```

```bash
docker ps
```

```bash
docker container ps
```

```bash
docker images
```

```bash
ps aux | grep httpd
```

```bash
ps aux | grep dockerd
```

```bash
ps aux | grep runc
```

## Lab 7 — Kontener interaktywny

Po pierwszym poleceniu pracujesz wewnątrz kontenera jako root. `exit` wraca na hosta.

```bash
docker container run -it --name moj_rocky rockylinux/rockylinux
```

```bash
yum install -y procps-ng iproute
```

```bash
hostname
```

```bash
ps aux
```

```bash
ip a
```

```bash
exit
```

```bash
docker ps
```

```bash
docker ps -a
```

## Lab 8 — Wykonywanie poleceń w kontenerze

Zastąp `af022d1baef2`, skrót `af` i nazwę `epic_noether` wartościami swojego kontenera Apache z laba 6.

```bash
docker container ps
```

```bash
docker exec -it af022d1baef2 id
```

```bash
docker exec -it af hostname
```

```bash
docker exec -it epic_noether uname -a
```

## Lab 9 — Inspekcja kontenera

Zastąp `epic_noether` nazwą swojego kontenera Apache.

```bash
docker container inspect epic_noether
```

## Lab 10 — Wybrane dane kontenera

Zastąp `epic_noether` nazwą swojego kontenera Apache, a `172.17.0.2` jego adresem IP z inspekcji.

```bash
docker container inspect --format '{{ (index .NetworkSettings.Networks "bridge").IPAddress }}' epic_noether
```

```bash
docker container inspect --format '{{ (index .NetworkSettings.Networks "bridge").Gateway }}' epic_noether
```

```bash
docker container inspect --format '{{ .Config.Env }}' epic_noether
```

```bash
docker container inspect --format '{{ .Created }}' epic_noether
```

```bash
docker container inspect --format '{{ .NetworkSettings.Ports }}' epic_noether
```

```bash
curl -s 172.17.0.2
```

## Lab 11 — Zmiana nazwy kontenera

Zastąp `af022d1baef2` identyfikatorem swojego kontenera Apache.

```bash
docker ps
```

```bash
docker container rename af022d1baef2 webserver
```

```bash
docker ps
```

## Lab 12 — Logi i procesy kontenera

Zastąp `7007` wartością PPID odczytaną z wyniku `docker container top webserver`.

```bash
docker container logs webserver
```

```bash
docker container top webserver
```

```bash
ps aux | grep 7007
```

## Lab 13 — Przekierowanie portu

Zastąp `172.17.0.2` adresem IP kontenera webserver. Pierwsze `curl localhost` może zgłosić błąd połączenia — port kontenera nie jest jeszcze opublikowany.

```bash
curl localhost
```

```bash
docker ps
```

```bash
docker container inspect --format '{{ (index .NetworkSettings.Networks "bridge").IPAddress }}' webserver
```

```bash
curl 172.17.0.2
```

Opcjonalnie: jeśli na hoście działa usługa Apache `httpd` zajmująca port 80, zatrzymaj ją z uprawnieniami administratora. Apache uruchomiony w kontenerze w labie 6 nie wymaga tego kroku.

```bash
sudo systemctl disable --now httpd
```

```bash
docker run -d -p 80:80 --name my_nginx nginx
```

```bash
docker ps
```

```bash
curl localhost
```

```bash
docker container inspect --format '{{ (index .NetworkSettings.Networks "bridge").IPAddress }}' my_nginx
```

```bash
docker container inspect --format '{{ .NetworkSettings.Ports }}' my_nginx
```

## Lab 14 — Katalogi Apache i Nginx

```bash
docker container exec webserver pwd
```

```bash
docker container exec my_nginx pwd
```

## Lab 15 — Obrazy, hostname i mapowanie portów

Przy `docker container prune` potwierdź operację wpisując `y`. Zastąp port `49153` portem kontenera web widocznym w `docker ps`. Po usunięciu zatrzymanych kontenerów usuń obraz `hello-world` pobrany w labie 4.

```bash
docker stop webserver
```

```bash
docker stop my_nginx
```

```bash
docker ps -a
```

```bash
docker container prune
```

```bash
docker ps -a
```

```bash
docker images
```

```bash
docker images -qa
```

```bash
docker rmi hello-world
```

```bash
docker search --limit 3 nginx
```

```bash
docker pull nginx
```

```bash
docker images
```

```bash
docker run -d --name webserver -h webserver -p 88:80 nginx
```

```bash
docker ps
```

```bash
curl -s localhost:88
```

```bash
docker exec -it webserver hostname
```

```bash
docker run -d --name web -h web -P nginx
```

```bash
docker ps
```

```bash
curl -s localhost:49153
```

```bash
docker exec -it web hostname
```

## Lab 16 — Zmiany wewnątrz kontenera i restart

Po `docker exec -it web bash` pracujesz wewnątrz kontenera. `exit` wraca na hosta. Zastąp port `49154` aktualnym portem z wyniku `docker port web`.

```bash
docker exec -it web bash
```

```bash
ls
```

```bash
service --status-all
```

```bash
cat /etc/nginx/nginx.conf
```

```bash
ls /etc/nginx/conf.d/
```

```bash
cat /etc/nginx/conf.d/default.conf
```

```bash
cat /usr/share/nginx/html/index.html
```

```bash
echo "witaj na moim serwerze www" > /usr/share/nginx/html/index.html
```

```bash
exit
```

```bash
docker ps
```

```bash
docker port web
```

```bash
docker restart web
```

```bash
docker ps
```

```bash
docker port web
```

```bash
curl -s localhost:49154
```

## Lab 17 — Tworzenie obrazu przez docker commit

Zastąp `172.17.0.4` adresem IP kontenera strona z inspekcji. Końcowe polecenie `docker container rm -f $(docker ps -aq)` usuwa wszystkie kontenery, także działające.

```bash
docker container commit web moja_www
```

```bash
docker images
```

```bash
docker image inspect moja_www
```

```bash
docker image inspect --format="{{.RootFS.Layers}}" nginx | wc -w
```

```bash
docker image inspect --format="{{.RootFS.Layers}}" moja_www | wc -w
```

```bash
docker run -d --name strona moja_www
```

```bash
docker ps
```

```bash
docker container inspect --format '{{ (index .NetworkSettings.Networks "bridge").IPAddress }}' strona
```

```bash
curl -s 172.17.0.4
```

```bash
docker container ps
```

```bash
docker ps -aq
```

```bash
docker container rm -f $(docker ps -aq)
```

```bash
docker ps
```

```bash
docker ps -a
```
