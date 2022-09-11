# Репозиторий файлы nginx.com

В данном репозитории находится **дефолтный** конфигурационный файл **nginx**

![nginx logo](https://www.logo.wine/logo/Nginx)



## Где взять файл

файл можно взять по нижеуказанной ссылке 
[default config](https://www.nginx.com/resources/wiki/start/topics/examples/full/)

## Структура конфигурационного файла

Конфигурационный файл **nginx** состоит из *модулей*, которые настраиваются *директивами*, указанными в конфигурационном файле. 
Директивы делятся на ==простые== и ==блочные==. 
1. Простая директива состоит из имени и параметров, разделённых пробелами, и оканчивается точкой с запятой (;). 
2. Блочная директива устроена так же, как и простая директива, но вместо точки с запятой после имени и параметров следует набор дополнительных инструкций, помещённых внутри фигурных скобок ({ и }). Если у блочной директивы внутри фигурных скобок можно задавать другие директивы, то она называется **контекстом** 

примеры: 
- events
- http
- server
- location

Директивы, помещённые в конфигурационном файле вне любого контекста, считаются находящимися в контексте **main**. 
- Директивы *events* и *http* располагаются в контексте **main**
- *server* — в **http**
- *location* — в **server**

> Часть строки после символа # считается комментарием.

## Раздача статического содержимого

Одна из **важных задач конфигурации nginx** — раздача файлов, таких как ==изображения== или ==статические HTML-страницы==. 

Рассмотрим пример, в котором в зависимости от запроса файлы будут раздаваться из разных локальных каталогов: `/data/www`, который содержит HTML-файлы, и `/data/images`, содержащий файлы с изображениями. Для этого потребуется отредактировать конфигурационный файл и настроить блок server внутри блока **http** с двумя блоками **location**.

Во-первых, создайте каталог `/data/www` и положите в него файл index.html с любым текстовым содержанием, а также создайте каталог `/data/images` и положите в него несколько файлов с изображениями.

Далее, откройте конфигурационный файл. Конфигурационный файл по умолчанию уже включает в себя несколько примеров блока *server*, большей частью закомментированных. Для нашей текущей задачи лучше закомментировать все такие блоки и добавить новый блок server:

```
http {
    server {
    }
}
```

В общем случае конфигурационный файл может содержать несколько блоков *server*, различаемых по портам, на которых они слушают, и по имени сервера. Определив, какой server будет обрабатывать запрос, **nginx** сравнивает URI, указанный в заголовке запроса, с параметрами директив **location**, определённых внутри блока *server*.

В блок server добавьте блок location следующего вида:

```
location / {
    root /data/www;
}
```

Этот блок **location** задаёт `“/”` в качестве префикса, который сравнивается с URI из запроса. Для подходящих запросов добавлением URI к пути, указанному в директиве **root**, то есть, в данном случае, к `/data/www`, получается путь к запрашиваемому файлу в локальной файловой системе. Если есть совпадение с несколькими блоками *location*, **nginx** выбирает блок с самым длинным префиксом. В блоке *location* выше указан самый короткий префикс, длины один, и поэтому этот блок будет использован, только если не будет совпадения ни с одним из остальных блоков *location*.

Далее, добавьте второй блок *location*:

```
location /images/ {
    root /data;
}
```

Он будет давать совпадение с запросами, начинающимися с `/images/` (`location /` для них тоже подходит, но указанный там префикс короче).

Итоговая конфигурация блока server должна выглядеть следующим образом:

```
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
    }
}
```

Это уже работающая конфигурация сервера, слушающего на стандартном порту `80` и доступного на локальном компьютере по адресу `http://localhost/`. В ответ на запросы, URI которых начинаются с `/images/`, сервер будет отправлять файлы из каталога `/data/images`. Например, на запрос `http://localhost/images/example.png` **nginx** отправит в ответ файл `/data/images/example.png`. Если же этот файл не существует, **nginx** отправит ответ, указывающий на **ошибку 404**. Запросы, URI которых не начинаются на `/images/`, будут отображены на каталог `/data/www`. Например, в результате запроса `http://localhost/some/example.html` в ответ будет отправлен файл `/data/www/some/example.html`.

Чтобы применить новую конфигурацию, запустите **nginx**, если он ещё не запущен, или отправьте сигнал reload главному процессу **nginx**, выполнив:

```
nginx -s reload
```

В случае если что-то работает не как ожидалось, можно попытаться выяснить причину с помощью файлов access.log и error.log из каталога `/usr/local/nginx/logs` или `/var/log/nginx`.


## Следующая таблица содержит иерархию конфигурационных файлов nginx.

| /etc/nginx/nginx.conf	 |  |  |  |
|:--|:--|:--|:--|
|  | /etc/nginx/conf.d/zz010_psa_nginx.conf	 |  |  |
|  |  | /etc/nginx/plesk.conf.d/server.conf	 |  |
|  |  |  /etc/nginx/plesk.conf.d/vhosts/@ <имя_домена> .conf -> /var/www/vhosts/system/ <имя_домена> /conf/last_nginx.conf|  |
|  |  |  | /var/www/vhosts/system/ <имя_домена> /conf/vhost_nginx.conf |
|  |  | /etc/nginx/plesk.conf.d/forwarding/ <имя_домена> .conf |  |


