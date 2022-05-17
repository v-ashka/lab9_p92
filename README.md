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

![image](https://user-images.githubusercontent.com/47278535/168814559-bac3477a-87ae-446f-8cca-ccb68f9978da.png)

![image](https://user-images.githubusercontent.com/47278535/168814580-9fc1a25f-b17b-457e-85ee-08955eaf2b71.png)


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

Do uruchomienia utworzonych w powyższym pliku kontenerów, użyto polecenia `docker compose up -d`
![image](https://user-images.githubusercontent.com/47278535/168814793-0f7b1f6a-b122-45da-81fe-87a513f1c61f.png)

Za pomocą polecenia `docker ps` można zauważyć że zostały uruchomione wszystkie kontenery zamieszczone w pliku docker compose
![image](https://user-images.githubusercontent.com/47278535/168814903-97275624-a4a8-4fe4-a72b-a1415bb6080c.png)

Dodatkowo utworzony docker compose zawiera dwie sieci, `frontend` oraz `backend`, które zostały utworzone, a za pomocą polecenia `docker network inspect` można dowiedzieć się na jakich adresach IP pracują utworzone konetnery

> Sieć backend
>![image](https://user-images.githubusercontent.com/47278535/168815153-b28a287b-1b29-4e36-9b05-cf35b33265fa.png)

> Sieć frontend
> ![image](https://user-images.githubusercontent.com/47278535/168815249-f8878828-39a0-492d-8803-cf94100ca6f7.png)

Jednym z założeń laboratorium było wyświetlenie strony index.php która posiadała informacje o wersji php, co przedstawia poniższy zrzut

> poprawne działanie serwera nginx+php
> ![image](https://user-images.githubusercontent.com/47278535/168815689-c4018d32-7f85-4fe5-a992-05fa94663c8d.png)

Jako że serwer nginx'a jest ustawiony na 2 sieci frontend i backend, jest możliwość połączenia się z dwóch adresów IP do powyższej strony.

Kolejnym punktem było poprawne uruchomienie się phpmyadmin, wraz z możliwością zalogowania się do bazy, poniższe zrzuty prezentują wykonanie tegoż punktu

> poprawne połączenie się z phpmyadmin
>  ![image](https://user-images.githubusercontent.com/47278535/168816264-a7286738-b1fc-45e1-b540-d8fb8f412f37.png)

> poprawne zalogowanie się do bazy danych
> ![image](https://user-images.githubusercontent.com/47278535/168816370-146a8605-5f80-4984-8368-6f566a5fe5ae.png)
> brak uprawnień dla użytkownika test, do tworzenia bazy danych
> ![image](https://user-images.githubusercontent.com/47278535/168816944-f82f036b-bd7d-419f-b501-5c0373bb4379.png)

Ze względu na to że podstawowy użytkownik nie ma uprawnień do utworzenia bazy, tworzenie zostało zaprezentowane na użytkowniku root, który jest adminem
![image](https://user-images.githubusercontent.com/47278535/168816753-f32d82d6-f455-48ee-b129-1bf7ffebae10.png)

### Generowanie reprezentacji graficznej docker compose
Jednym ze sposób wygenerowania graficznej reprezentacji utworzonego docker compose, jest darmowe narzędzie `docker-compose-viz`, ([Link tutaj](https://github.com/pmsipilot)), to narzędzie pozwala na utworzenie pliku graficznego całej struktury docker compose'a

![image](https://user-images.githubusercontent.com/47278535/168817780-db8387dc-4921-4f88-8271-ef600584ede5.png)



