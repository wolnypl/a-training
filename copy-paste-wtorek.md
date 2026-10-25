## Lab 18 — Budowanie obrazu z Dockerfile

Wykonuj jako admin na hoście. Zawartość bloków `dockerfile`, `sql` i `yaml` w kolejnych labach wklejaj do wskazanego pliku w edytorze, a nie do powłoki. Możesz zastąpić `vim` edytorem `nano`. Po zapisaniu pliku zamknij edytor i wykonaj następne polecenie.

Zainstaluj dodatek Buildx, jeśli nie został zainstalowany razem z Dockerem.

```bash
sudo dnf install -y docker-buildx-plugin
```

```bash
cd ~
```

```bash
mkdir -p user
```

```bash
cd user
```

```bash
echo "Dodanie pliku do obrazu" > dane.txt
```

```bash
vim Dockerfile
```

Zawartość pliku `Dockerfile`:

```dockerfile
FROM rockylinux/rockylinux:9

RUN useradd admin && echo 'admin:altkom' | chpasswd
RUN dnf -y update && dnf -y install cifs-utils && dnf clean all

ADD dane.txt /tmp/

CMD ["cat", "/tmp/dane.txt"]
```

```bash
cat Dockerfile
```

```bash
docker build -t rocky:user .
```

```bash
docker run rocky:user
```

## Lab 19 — Obraz z serwerem Apache

```bash
cd ~
```

```bash
mkdir -p www
```

```bash
cd www
```

```bash
echo "Strona www z Dockerfile" > index.html
```

```bash
vim Dockerfile
```

Zawartość pliku `Dockerfile`:

```dockerfile
FROM rockylinux/rockylinux:9

LABEL org.opencontainers.image.authors="altkom <root@localhost>"

RUN dnf -y install httpd && dnf clean all && mkdir -p /run/httpd

COPY index.html /var/www/html/

EXPOSE 80

CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
```

```bash
cat Dockerfile
```

```bash
docker build -t rocky:webserver .
```

Lab 17 powinien wcześniej usunąć kontener o nazwie webserver. Port 8081 na hoście musi być wolny.

```bash
docker run -d --name webserver -p 8081:80 rocky:webserver
```

```bash
curl -s localhost:8081
```

## Lab 20 — Obraz MySQL z początkowymi danymi

```bash
cd ~
```

```bash
mkdir -p mysql
```

```bash
cd mysql
```

```bash
vim table.sql
```

Zawartość pliku `table.sql`:

```sql
CREATE TABLE altkom (imie VARCHAR(30), specjalizacja VARCHAR(30));
INSERT INTO altkom (imie, specjalizacja) VALUES
    ('Aneta', 'Docker'),
    ('Pawel', 'Docker'),
    ('Maciej', 'Docker'),
    ('Michal', 'Docker'),
    ('Michal K.', 'Docker');
```

```bash
cat table.sql
```

```bash
vim Dockerfile
```

Zawartość pliku `Dockerfile`:

```dockerfile
FROM mysql:8.4

LABEL org.opencontainers.image.authors="altkom <root@localhost>"

ENV MYSQL_DATABASE=altkom

COPY table.sql /docker-entrypoint-initdb.d/
```

```bash
cat Dockerfile
```

```bash
docker build -t training:db .
```

Hasło roota podaj przy uruchomieniu. W tym ćwiczeniu jest to `altkom`. Skrypt SQL wykonuje się tylko przy inicjalizacji pustego katalogu danych, nie przy każdym restarcie kontenera.

```bash
docker run -d --name bazaDB -e MYSQL_ROOT_PASSWORD=altkom training:db
```

```bash
docker ps
```

```bash
docker logs -f bazaDB
```

Poczekaj na końcowy komunikat `ready for connections` po wykonaniu `table.sql` i zakończeniu inicjalizacji. Naciśnij Ctrl+C — kończysz obserwowanie logów, kontener nadal działa.

```bash
docker exec -it bazaDB /bin/bash
```

Wewnątrz kontenera:

```bash
mysql -u root -p
```

Wpisz hasło `altkom`. Następne polecenia wykonuj w konsoli MySQL:

```sql
SHOW DATABASES;
```

```sql
USE altkom;
```

```sql
SHOW TABLES;
```

```sql
SELECT imie, specjalizacja FROM altkom;
```

```sql
exit;
```

W powłoce kontenera wróć na hosta:

```bash
exit
```

```bash
cd ~
```

```bash
docker ps -q
```

Poniższe sprzątanie zatrzymuje wszystkie działające kontenery, a następnie usuwa wszystkie zatrzymane kontenery. Przy `prune` potwierdź operację wpisując `y`. Jeśli lista działających kontenerów jest pusta, pomiń polecenie `docker stop`.

```bash
docker stop $(docker ps -q)
```

```bash
docker ps -a
```

```bash
docker container prune
```

## Lab 21 — Udostępnianie katalogu hosta kontenerom

Wykonuj cały lab jako ten sam użytkownik admin. Jest to bind mount katalogu hosta. Opcja `:z` umożliwia współdzielenie tego katalogu przez kontenery również przy aktywnym SELinux.

```bash
cd ~
```

```bash
mkdir -p data
```

```bash
ls
```

```bash
docker run -ti --name vol1 -v "$HOME/data:/DATA:z" busybox
```

Wewnątrz kontenera vol1:

```bash
ls
```

```bash
ls /DATA
```

```bash
ls /etc
```

```bash
cp /etc/hosts /DATA/
```

```bash
ls /DATA
```

```bash
cat /DATA/hosts
```

```bash
exit
```

Na hoście:

```bash
ls data
```

```bash
cat data/hosts
```

```bash
docker run -ti --name vol2 -v "$HOME/data:/DATA:z" busybox
```

Wewnątrz kontenera vol2:

```bash
ls
```

```bash
ls /DATA
```

```bash
cat /DATA/hosts
```

```bash
mkdir /DATA/nowy_katalog
```

```bash
exit
```

Na hoście:

```bash
ls data
```

## Lab 22 — Administracja woluminami

```bash
docker volume ls
```

```bash
docker volume create myvol1
```

```bash
docker volume ls
```

```bash
docker volume inspect myvol1
```

```bash
docker container run -ti --rm -v myvol1:/data1 busybox
```

Wewnątrz pierwszego kontenera:

```bash
ls
```

```bash
touch /data1/plik1
```

```bash
echo "plik kontrolny w myvol1" > /data1/plik1
```

```bash
cat /data1/plik1
```

```bash
exit
```

Na hoście:

```bash
docker volume inspect myvol1
```

```bash
docker container run -ti --rm -v myvol1:/data2 --name samokasujacy busybox
```

Wewnątrz drugiego kontenera:

```bash
ls /data2
```

```bash
cat /data2/plik1
```

```bash
mkdir /data2/nowy_katalog
```

```bash
exit
```

Na hoście zastąp `/var/lib/docker/volumes/myvol1/_data` wartością `Mountpoint` odczytaną z `docker volume inspect myvol1`, jeśli jest inna.

```bash
sudo ls /var/lib/docker/volumes/myvol1/_data/
```

```bash
sudo cat /var/lib/docker/volumes/myvol1/_data/plik1
```

```bash
docker volume ls
```

To polecenie usuwa wszystkie nieużywane woluminy, również nazwane — w tym myvol1 i ewentualne pozostałości danych MySQL z laba 20. Potwierdź operację wpisując `y`.

```bash
docker volume prune --all
```

Kontenery vol1 i vol2 są już zatrzymane przez `exit`, a kontenery z `--rm` zostały automatycznie usunięte. Usuń zatrzymane kontenery, potwierdzając `y`.

```bash
docker container prune
```

## Lab 23 — Compose i aplikacja z wielorybkami

Wykonuj jako admin na hoście. Jeśli dodatek Compose nie jest jeszcze zainstalowany:

```bash
sudo dnf install -y docker-compose-plugin
```

```bash
docker compose version
```

```bash
cd ~
```

```bash
mkdir -p wielorybki
```

```bash
cd wielorybki
```

```bash
vim docker-compose.yml
```

Zawartość pliku `docker-compose.yml`:

```yaml
services:
  redis:
    image: redis:alpine
    volumes:
      - redis_data:/data
    restart: always
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 20

  mobycounter:
    depends_on:
      redis:
        condition: service_healthy
    image: russmckendrick/moby-counter
    ports:
      - "8080:80"
    restart: always

volumes:
  redis_data:
```

```bash
cat docker-compose.yml
```

```bash
docker compose config -q
```

```bash
docker compose up
```

Otwórz w przeglądarce `http://localhost:8080` na hoście Dockera albo `http://ADRES_IP_HOSTA:8080` z innego komputera. Kliknij kilka razy na stronie. Zatrzymaj aplikację klawiszami Ctrl+C. Przed kolejnym poleceniem poczekaj na powrót do powłoki.

```bash
docker compose down
```

## Lab 24 — WordPress z Compose

```bash
mkdir -p ~/wordpress
```

```bash
cd ~/wordpress
```

```bash
vim docker-compose.yml
```

Zawartość pliku `docker-compose.yml`:

```yaml
services:
  db:
    image: mysql:8.4
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: altkom
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    healthcheck:
      test: ["CMD-SHELL", "MYSQL_PWD=$$MYSQL_PASSWORD mysql --protocol=TCP -h 127.0.0.1 -u$$MYSQL_USER -D $$MYSQL_DATABASE -e 'SELECT 1'"]
      interval: 5s
      timeout: 5s
      retries: 30
      start_period: 30s

  wordpress:
    depends_on:
      db:
        condition: service_healthy
    image: wordpress:latest
    ports:
      - "8000:80"
    volumes:
      - wordpress_data:/var/www/html
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress

volumes:
  db_data:
  wordpress_data:
```

```bash
cat docker-compose.yml
```

```bash
docker compose config -q
```

```bash
docker compose up -d
```

Otwórz w przeglądarce `http://localhost:8000` na hoście Dockera albo `http://ADRES_IP_HOSTA:8000` z innego komputera. Poczekaj na inicjalizację WordPressa. Jeśli używasz katalogu z istniejącymi danymi MySQL 5.7, nie uruchamiaj na nich bezpośrednio MySQL 8.4 — ten przykład wymaga nowego woluminu lub uprzedniej migracji danych.

```bash
docker compose ps
```

```bash
docker compose images
```

```bash
docker compose top
```

```bash
docker compose logs -f
```

Naciśnij Ctrl+C, aby zakończyć obserwowanie logów. Następne polecenie usuwa kontenery i woluminy projektu, w tym bazę i pliki WordPressa.

```bash
docker compose down -v
```

## Lab 25 — Sieci Dockera

Wykonuj na hoście. Obraz moja_www powinien pozostać po labie 17.

```bash
docker network ls
```

```bash
docker network inspect bridge
```

```bash
docker network create --driver bridge www_lan
```

```bash
docker network ls
```

```bash
docker network inspect www_lan
```

```bash
docker run -d --network www_lan --name www moja_www
```

```bash
docker ps
```

```bash
docker container inspect --format '{{ (index .NetworkSettings.Networks "www_lan").IPAddress }}' www
```

```bash
docker network inspect www_lan
```

```bash
docker run -it --network www_lan alpine
```

Wewnątrz kontenera Alpine sprawdź adres i komunikację z serwerem www. Nazwa www jest rozwiązywana przez DNS sieci www_lan.

```bash
ip a
```

```bash
ping -c 3 www
```

```bash
wget -qO- http://www
```

```bash
exit
```

Na hoście:

```bash
docker run -d --network www_lan nginx
```

```bash
docker network inspect www_lan
```

Sprawdź izolację sieci z kontenera w domyślnej sieci bridge:

```bash
docker run -it alpine
```

Wewnątrz kontenera zastąp `172.24.0.2` adresem kontenera www odczytanym wcześniej. Przy domyślnej konfiguracji Dockera ping między odrębnymi sieciami bridge powinien się nie powieść.

```bash
ip a
```

```bash
ping -c 3 -W 2 172.24.0.2
```

```bash
exit
```
