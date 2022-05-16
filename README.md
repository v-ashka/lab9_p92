# Laboratorium 9 

## Zadanie P9.2
>
> Na poprzednich laboratoriach budowane były obrazy kontenerów z serwerem Apache, PHP.
>W tym zadaniu należy zbudować prosty plik docker-compose.yml, który pozwoli na uruchomienie
>znanej z innych zajęć, usługi LEMP wraz z phpMyAdmin. Stack LEMP składa się z następujących
>usług składowych:
>
>---- L – dla Linux;
>---- E – dla Nginx;
>---- M – dla MySQL;
>---- P – dla PHP.
>Wobec tego projekt powinien zawierać cztery kontenery (usługi):
>- jeden kontener dla Nginx,
>- jeden kontener dla PHP (PHP-FPM) https://php-fpm.org/,
>- jeden kontener dla MySQL,
>- jeden kontener dla phpMyAdmin.
> Założenia dla usługi:
>- serwery są budowane zgodnie z dokumentacją obrazów bazowych dostępnych na DockerHub i
> umieszczonych tam plików Dockerfile (zbudowanie własnych obrazów a nie wykorzystanie z
> gotowych obrazów będzie wyżej oceniane).
> - serwery PHP, MySQL, phpMyAdmin są przyłączone do sieci backend a Nginx do backend oraz
> frontend. Nginx ma wystawiony na świat zewnętrzy port 6666,
> - serwer Nginx ma wyświetlać stronę startową php (index.php),
> - serwer phpMyAdmin ma być dostępny na porcie 6001 i powinno być możliwe zalogowanie się do
> niego i założenie testowej bazy. 

Została stworzona następująca struktura katalogów
```
/p92
├── docker-compose.yml
├── .env
├── nginx
│   ├── default.conf
│   └── Dockerfile
└── www
    └── html
        └── index.php
```

W pliku `nginx/default.conf` znajdują się ustawienia nginx'a, umożliwiające poprawne uruchomienie się pliku `index.php`

Plik `Dockerfile`, zawiera instrukcje COPY, która kopiuje stworzoną konfigurację do kontenera nginxa, zaś w pliku `.env` znajdują się hasła niezbędne do zalogowania się do tworzonej bazy danych.

Utworzony docker-compose wygląda następująco:

```yml
#ustawienie wersji compose'a
version: "3"


services:
#serwer nginx'a
  nginx:
    # builduje dockefile znajdującego się w podanym folderze
    build: ./nginx/
    # ustawia nazwę kontenera
    container_name: nginx-container
    #ustawienie portu
    ports:
      - 6666:80
    #ustawienie wolumenu który wskazuje jaki katalog ma być wyświetlony przez ngixna
    volumes:
      - ./www/html/:/var/www/html
    # dołączenie do dwóch sieci
    networks:
      - backend
      - frontend
  # serwis php
  php:
    # obraz php-fpm
    image: php:7.0-fpm
    # nazwanie kontenera
    container_name: php-container
    # expose na port 9000, podobny port znajduje się w pliku default.conf
    expose:
      - 9000
    #ustawienie wolumenu który wskazuje jaki katalog ma być wyświetlony przez php
    volumes:
      - ./www/html/:/var/www/html
    # podłączenie do sieci backend
    networks:
      - backend
  # serwis mysql
  mysql:
    image: mysql:latest
    # ustawienie zmiennych środowiskowych
    environment:
      # zmienne w ${} wskazują na plik .env
        MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASS}
        # nazwa usera
        MYSQL_USER: test
        # haslo usera
        MYSQL_PASSWORD: ${MYSQL_PASS}
    # podłączenie do sieci backend
    networks:
      - backend
  # serwis phpmyadmin
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    # ustawienie zmiennych środowiskowych
    environment:
      # określenie hosta MySQL, wskazuje na usługę mysql 
      PMA_HOST: mysql
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASS}
      # ustawienie portu
    ports:
      - 6001:80
    # ustawienie sieci backend
    networks:
      - backend

# utworzone sieci
networks:
  frontend:
  backend:

```