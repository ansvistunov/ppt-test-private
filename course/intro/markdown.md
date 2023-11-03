# Введение
---
### Определения
 - «Распределенная система это набор независимых узлов (компьютеров), которые представляются пользователю как единая система.»<!-- .element: class="size"  -->
 - «Распределенная система это собрание независимых компьютеров соединенных сетью с программным обеспечением, обеспечивающим их совместное функционирование.»<!-- .element: class="size"  -->
 - «Система, состоящая из набора двух или более независимых узлов, которые координируют свою работу посредством синхронного или асинхронного обмена сообщениями.»<!-- .element: class="size"  -->
 - «Система, чьи компоненты размещены на различных узлах, взаимодействующие и управляемые только посредством передачи сообщений.»<!-- .element: class="size"  -->
 - «...система нескольких автономных вычислительных узлов, взаимодействующих для выполнения общей цели.»<!-- .element: class="size"  -->
 - «…я не могу объяснить, что такое распределенная система, но узнаю ее как только мне ее покажут»<!-- .element: class="size"  -->
---
### Определения
 - В рамках этого курса мы будем рассматривать такие системы, компоненты которых *выполняются на различных узлах* и взаимодействуют между собой посредством какой-либо *сети передачи данных*
---
### Очевидные последствия
 - Дополнительный компонент с *низкой* надежностью - сеть
    - Значительная задержка при передаче даных (латентность при передаче)
    - Незначительная (по сравнению с RAM) пропускная способность (для RAM десятки ГБайт\с, для сети Гбиты\с)
    - Проблемы безопасности в сети (возможность перехвата и подмены трафика)
    - топология сети может изменяться во время работы приложения
 - Любое распрделенное приложение - по определению параллельное
    - гонки потоков
    - необходимость в синхронизации
---
### Очевидные последствия
 - Сложности с развертыванием приложения
     - Как обновлять приложение, состоящее из десятков компонентов
     - Части приложения могут разворачиваться на "чужих" площадках - вне периметра безопасности
 - Сложности с эксплуатацией и обнаружением сбоев
     - Как понять, что какие-то компоненты распределенного приложения вышли из строя
 - Нет "глобального" состояния системы (нет ни одного процесса в распределенной системе, который бы знал текущее глобальное состояние системы)
 - Строго говоря - нет "глобального" времени (ограниченная точность синхронизации часов) 
---
### Очевидные последствия
 - Распределенные системы сложно проектировать
 - ... сложно разрабатывать
 - ... сложно эксплуатировать

 Зачем они вообще нужны?
---
### Мотивация
 - Естественное разделение. Существуют ситуации, в которых распределенная система - единственная возможность
     - почтовый клиент и почтовый сервер
     - браузер и HTTP-сервер
     - банкомат и процессинговая система банка
     - касса в магазине и процессинг банка\система магазина
     - ...
 - Большинство ИТ-систем, с которыми пользователь сегодня сталкивается - распределенные. Потому что пользователь и выполнение его сервиса находятся в разных местах    
---
### Мотивация
 - Повышение производительности
    - Зачастую один узел системы не может справиться с вычислительной нагрузкой. Увеличивать мощность узла нежелательно (например, дорого). В систему добавляют допонительные узлы
    - Для обеспечения *масштабируемости* система должна быть специальным образом спроектирована. Например, при ее создании могут использоваться специализированные шаблоны или целые библиотеки
    - Вы уже знакомы с таким подходом: MPI как раз решает эту задачу 
---
### Мотивация
 - Отказоустойчивость
    - Отдельные (или все) компоненты системы могут быть дублированы на разных (независимых) узлах. Тогда выход из строя одного (или нескольких - зависит от количества запущенных дубликатов) узла не приводит к прекращению работы системы
    - Часто используется в варианте, когда происходит не дублирование компонентов, а их *перенос* с аварийного узла на рабочий. При этом перерыв в сервисе есть, но он минимальный
    - Часто используется совместно с мероприятиями по обеспечению масштабируемости
---
### Мотивация
 - Другие, чаще всего "экономические" причины : если ваша систем разбита на небольшие компоненты, между которыми четко определен интерфейс, то
    - можно повысить T2M (время от идеи до внедрения) 
        - в такие компоненты можно быстро вносить изменения
        - развертывать при изменениии можно отдельные компоненты, а не все приложение
    - можно повысить продуктивность разработки 
        - разные компоненты могут быть написны на разных ЯП (наиболее подходящих для решения задачи), использовать разные СУБД
        - разные компоненты могут разрабатывать разные команды разработчиков
---
### Общие соображения
 - В зависимости от того, чем именно вызвана необходимость создания распределенной системы, к ее компонентам могут предъявляться разные требования. Однако, можно сформулировать ряд общих требований, видимо подходящих для большинства систем:
     - При проектировании межмодульного интерфейса необходимо внимательно следить за объемом передаваемых данных. Если данных *слишком много*, возможно, вы неправильно провели границы между модулями
     - При прочих равных отдавать предпочтение массированным операциям передачи данных. Передать массив значений *выгоднее*, чем отдельное значение
---
### Общие соображения
 - Старайтесь избегать сильной связанности между компонентами. Компоненты должны "знать" только о тех компонентах, которые им нужны для работы (с которыми они взаимодействуют)
 - Инкапсулируйте компоненты - предоставляйте интерфейс(API), за которым скрыта вся внутренняя логика работы
 - Если есть возможность реализовать компонент без хранения состояния (stateless) - сделайте это
 - Если заботитесь о надежности - позаботьтесь, чтобы в системе не было единой точки отказа (одного компонента, выход из строя которого приведет к краху всей системы)    
---
### Общие соображения
В настоящее время, из соображений максимальной независимости компонентов друг от друга, принят подход, при котором все взаимодействие между компонентами осуществляется посредством API компонента. Это означает, что компонент должен полностью отвечать за сохранение своего состояния (если оно есть), т.е. каждый компонент имеет своб собственную СУБД. Это в корне отличается от привычного подхода, при котором данные всей системы хранятся согласованно и централизованно <!-- .element: class="left small_font" -->

  - Плюсы такого подхода: <!-- .element: class="size"  -->
     - каждый компонент выбирает СУБД и схему данных оптимальную для решения своей задачи <!-- .element: class="size"  -->
     - СУБД и схема данных может быть в любой момент изменена, если это не сказывается на API <!-- .element: class="size"  -->
     - нет единой точки отказа (центральной СУБД) <!-- .element: class="size"  -->
     - нет единого "бутылочного горлышка" производительности (центральной СУБД) <!-- .element: class="size"  --> 
 - Минусы <!-- .element: class="size"  -->
     - необходимо самостоятельно решать проблему согласованности данных для СУБД разных компонент, что может оказаться очень сложной задачей <!-- .element: class="size"  -->
     - у нас теперь нет "простых" транзакций в общей СУБД, любая транзакция становится распределенной <!-- .element: class="size"  -->
---
### Микросервисная архитектура
 - набор правил и соглашений для компонентов распределенной системы
     - сервис решает бизнес-задачу (группировка по бизнес-задачам)
     - сервис **маленький** (разрабатывается одной командой, и его способен понять один человек) 
     - сервисы слабосвязаны (поэтому их при желании можно повторно использовать)
     - используется децентрализованное управление сервисами
     - используется децентрализованное управление данными (у каждого сервиса свое хранилище)
     - используется автоматизация развертывания и мониторинга
---
### Микросервисная архитектура
 - Плюсы
     - маленькие сервисы проще создавать и сопровождать
     - упрощается обновление системы - маленькте автономные сервисы могут быть обновлены независимо
     - для каждого сервиса - подходящая ему технология (ЯП, СУБД, ...)
     - высокая стабильность - отказ сервиса может быть обнаружен и исправлен (в т.ч. автоматически)
     - возможность масштабирования как средство увеличения производительности и надежности
 
---
### Микросервисная архитектура
 - Минусы
     - сервисы маленькие -> много взаимодействий. может страдать производительность -> производительность (и надежность) сети критически важна
     - сервисов много -> нужно много оборудования (узлов) для их развертывания. зачастую *больше* чем для развертывания монолита
     - много сервисов и оборудования - критическая зависимость от средств автоматизации управления
     - много API. нужны способы его описания 
     - при проектировании нужно учитывать много особенностей - асинхронный характер взаимодействия, то, что отдельные сервисы могут быть недоступны и т.д. и т.п. 
---
### Теорема CAP
в любой реализации распределённых вычислений возможно обеспечить не более двух из трёх следующих свойств (это эвристическое утверждение!):<!-- .element: class="left" -->
 - согласованность данных (англ. consistency) — во всех вычислительных узлах в один момент времени данные не противоречат друг другу;
 - доступность (англ. availability) — любой запрос к распределённой системе завершается корректным откликом, однако без гарантии, что ответы всех узлов системы совпадают;
 - устойчивость к разделению (англ. partition tolerance) — расщепление распределённой системы на несколько изолированных секций не приводит к некорректности отклика от каждой из секций.
---
### Теорема CAP
![CapTeorem](https://i0.wp.com/proselyte.net/wp-content/uploads/2022/03/CAP_diagram.png)<!-- .element: width="40%" -->

https://i0.wp.com/proselyte.net/wp-content/uploads/2022/03/CAP_diagram.png<!-- .element: class="copyright-reference"  -->

Таким образом, при создании распределенной системы приходится идти на компромисы и выбирать какой-то из вариантов<!-- .element: class="left small_font" -->
---
### Модели архитектура
  - Модель архитектуры отвечает на вопрос:
     - какие компоненты составляь нашу распределенную систему (и какие у них роли)
     - как компоненты взаимодействуют между собой
  - Выбор архитектуры при проектировании - критически важен. Он влияет на будущие эксплуатационные свойства 
  - Самой простой моделью архитектуры является модель "клиент-сервер"   
---
---
### Клиент-сервер
 - Клиент: процесс, желающий получить доступ к данным, использует ресурсы и/или выполняет действия на удаленном узле
 - Сервер: процесс, управляющий данными и всеми другими разделяемыми ресурсами, обеспечивающий клиентам доступ к ресурсам и  производящий вычисления
 - Взаимодействие: пары запрос / результат
 - Пример
     - http-сервер: клиент (броузер) запрашивает страницу, сервер поставляет страницу
     - клиент посылает SQL-запрос на сервер СУБД, сервер его выполняет и возвращает ответ
---
### Клиент-сервер (мобильный код)
 - Мобильный код: код, посланный клиенту, чтобы выполнить его же задачу
![MobileCode](../img/CSMobileCode.png)
---
### Клиент-сервер (мобильный код)
 - Выполнение программы (код + данные), которая перемещается между узлами в сети
     - Выполняет автономную задачу обычно под управлением некоторого другого процесса
     - Имеет внутреннее знание и цели
 - Преимущество: всюду локальный доступ
     - Снижает затраты на коммуникации
 - Потенциальная угроза безопасности
     - Ограниченная применимость
 - Примеры: сбор данных из многих источников, установка программ, программы типа червей (электронная почта)
---
### Модель с прокси-сервером
![Proxy](../img/CSProxy.png)
---
### Модель с прокси-сервером
 - Кэш: «близкая» копия, наиболее часто используемых данных
     - ЗНАЧИТЕЛЬНО повышает производительность большинства приложений
     - Но требует усилий по поддержанию когерентности
 - Прокси-сервер: разделяемый кэш ресурсов
     - Еще сложнее, чем простой Кэш…
     - Обычно используется (и хорошо подходит) для доступа к мало меняющимся или *неважным* данным
---
### Модель «пул серверов» (сервис)
![Service](../img/CSService.png)
---
### Модель «пул серверов» (сервис) 
 - Услуги могут обеспечиваться многими серверами 
 - Распределенные между серверами объекты
 - Реплицированные объекты
     - Увеличение производительности, доступности и отказоустойчивости
 - Но требуют координации копий / консистентности представления
 - Например, высокодоступные серверы (порталы, диллинговые центры), информационные службы
 - Серверы, обслуживающие распределенную БД
---
### Часто используемые модули. util
 - Util.inspect – вывод объекта в консоль (аналог toString в Java)
 - Util.format – аналог printf (форматный вывод)
 - Util.inherits – устанавливает отношение «наследования» между объектами JavaScript
---
### Часто используемые модули. concole
 - Глобальная (!!!) переменная
 - Console.log выводит в stdout
 - Console.error выводи в stderr
 - Console.trace выводит stacktrace в поток ошибок
---
### Часто используемые модули. events
Объект EventEmitter позволяет:<!-- .element: class="left"  -->
 - Определять обработчики (функции) для событий
    - Для каждого события может быть определено несколько обработчиков
    - Обработчики выполняются в порядке их определения
 - Генерировать события
 - Для каждого события можно получить список его обработчиков
 - Событие error генерирует исключение (если для него не определен обработчик)
---
### Часто используемые модули. events
```js
const EE =require("events").EventEmitter;
const server = new EE();

server.on('request', function(request){
    request.approved = true;
});

server.on('request', function(request){
    console.log(request);
});

server.emit('request',{from:'client1'});
server.emit('request',{from:'client2'});
```
---
### Часто используемые модули. http
Позволяет<!-- .element: class="left"  -->
 - Организовать HTTP сервер
     - Связать его с нужным портом (и интерфейсом)
     - Определить обработчики для событий (например, событие request)
     - Получать в обработчиках объект - запрос клиента и объект, для формирования ответа сервера
 - Выполнить HTTP – запрос
     - И обработать ответ 
---
### Часто используемые модули. http
 ```js
const http = require("http");

const server = new http.Server();

server.listen(4848, "localhost");

server.on('request',function(req,res){
    res.end("Hello world ");
});
```
---
### Часто используемые модули. http (клиент)
 ```js
http.get('http://www.unn.ru', function(response) {
    //console.log('1');
    let body = '';
    response.on('data', function(d) {
        body += d;
        //console.log('2');
    });
    response.on('end', function() {
        //console.log('3');
        console.log(body);
    });
});
```
---
### Часто используемые модули. url
 URL позволяет<!-- .element: class="left"  -->
 - Сформировать URL из отдельных элементов (протокол, адрес, порт и т.д.)
 - Извлечь из строки URL отдельные элементы:
     - url.parse(<строка URL>, [сформировать объект])
     - http://localhost:4848/echo?message=Hello
```js
Url {protocol: null,
    slashes: null,
	auth: null,
	host: null,
	port: null,
	hostname: null,
	hash: null,
	search: '?message=Hello',
	query: [Object: null prototype] { message: 'Hello' },
	pathname: '/echo',
	path: '/echo?message=Hello',
	href: '/echo?message=Hello'}
```
---
### Часто используемые модули. url
```js 
const http = require("http");
const url = require("url");
const server = new http.Server();
server.listen(4848, "localhost");

server.on('request',function(req,res){
    let urlParsed = url.parse(req.url, true);
    console.log(req.method+" "+req.url);
    console.log(urlParsed);
    if (urlParsed.pathname == "/echo" &&  urlParsed.query.message){
        res.end(urlParsed.query.message);
    }else{
        res.status = 404;
        res.end("Page not found");
    };
});
```
---
### Часто используемые модули. fs
FS позволяет<!-- .element: class="left"  -->
 - Выполнять операции с элементами файловой системы
     - Читать данные из файлов
     - Создавать файлы
     - Записывать данные в файлы
     - Удалять файлы
    - Переименовывать файлы
 - Операции с файлами могут быть
     - Синхронными
     - Асинхронными 
---
### Часто используемые модули. fs
 ```js 
const http = require("http");
const url = require("url");
const fs = require("fs");
const server = new http.Server();
server.listen(4848, "localhost");
server.on('request',function(req,res){
    let urlParsed = url.parse(req.url, true);
    console.log(urlParsed);
    let data;
    if (req.url == "/") {
        data = fs.readFileSync("index.html");
        res.end(data);
    }else{
        try {
            data = fs.readFileSync("." + urlParsed.path);
            res.end(data);
        }catch(error){
            res.status = 404;
            res.end("Page not found");
        }
    }
});
```
---
### Часто используемые модули. fs
 ```js 
const http = require("http");
const url = require("url");
const fs = require("fs");
const server = new http.Server();
server.listen(4848, "localhost");
server.on('request',function(req,res){
    const readFile = function (err, info) {
        if (err) {
            res.status = 404;
            res.end("Page not found");
        } else
            res.end(info);
    };
    let urlParsed = url.parse(req.url, true);
    let data;
    if (req.url == "/") {
        data = fs.readFile("index.html", readFile);
    }else{
        data = fs.readFile("." + urlParsed.path, readFile);
    }
});
```
---
### Часто используемые модули. fs
 ```js [17-25]
server.on('request', function (req, res) {
    let urlParsed = url.parse(req.url, true);
    let data;
    if (req.method === "GET") {
        if (req.url == "/") {
            data = fs.readFileSync("index.html");
            res.end(data);
        } else {
            try {
                data = fs.readFileSync("." + urlParsed.path);
                res.end(data);
            } catch (error) {
                res.status = 404;
                res.end("Page not found");
            }
        }
    } else if (req.method === "POST") {
        let body = '';
        req.on('data', chunk => {
            body += chunk.toString(); // convert Buffer to string
        });
        req.on('end', () => {
            console.log(body);
        });
    }
});
```
---
### Часто используемые модули. fs
 - вывод предыдущей программы:

 ```
------WebKitFormBoundaryc4cSmzSsiq2jHW7Q
Content-Disposition: form-data; name="fileToUpload"; filename="test.json"
Content-Type: application/json

{
  "Hello":"привет",
  "test":"Тест"
}
------WebKitFormBoundaryc4cSmzSsiq2jHW7Q--
```
 - т.е. нам как-то нужно суметь разобрать multipart/form-data. 
 - Можно делать это вручную, или можно использовать готовые модули (их много), например **formidable** или **multer**. 
 - Как правило они не являются стандартными и требуют установки
---
### fs + formidable
 ```js [1-33|4|23-31]
const http = require("http");
const url = require("url");
const fs = require("fs");
const formidable  =  require('formidable');
const server = new http.Server();
server.listen(4848, "localhost");
server.on('request', async function (req, res) {
    let urlParsed = url.parse(req.url, true);
    let data;
    if (req.method === "GET") {
        if (req.url == "/") {
            data = fs.readFileSync("index.html");
            res.end(data);
        } else {
            try {
                data = fs.readFileSync("." + urlParsed.path);
                res.end(data);
            } catch (error) {
                res.status = 404;
                res.end("Page not found");
            }
        }
    } else if (req.method === "POST") {
        const form = new formidable.Formidable({ multiples: true });
        let fields;
        let files;
        [fields, files] = await form.parse(req);
        fs.rename(files['fileToUpload'][0].filepath, './upload/'+files['fileToUpload'][0].originalFilename, (err)=>{
            if (err) console.log(`renamed. error is ${err}`);
        })
        res.end("Success upload")
    }
});
```
---
### Часто используемые модули. Express
 - Модуль Express не является “стандартным”
     - Требует установки (npm install express)
 - Обладает богатой функциональностью (всю рассматривать не будем), в частности
     - Позволяет определить обработчики для url, body (автоматическое преобразование в объекты JS)
     - Позволяет определить автоматическую обработку для статических элементов (файлы html, css, изображения, …)
     - Позволяет определить протоколы и обработчики для доступа к различным ресурсам
     - И многое, многое другое…
---
### Часто используемые модули. Express
```js 
const express = require("express");
//подключили модуль
const app = express();
//инициализировали

app.listen(3000);
//связали с портом и запустили сервер
```
---
### Часто используемые модули. Express
 - Маршрутизация
 - Для определения ресурсов и методов для доступа к ним используется следующий механизм:
 ```js 
 app.METHOD(ENDPOINT, HANDLER)
 ```
     - Где METHOD определяет протокол доступа (get, put, delete, post, all)
     - ENDPOINT определяет имя ресурса. Может быть задан в виде регулярного выражения (app.get('/ab*cd', …..);)
     - HANDLER – функция для обработки ресурса. Принимает request и response
---
### Часто используемые модули. Express
```js 
var express = require("express");
var app = express();
app.get("/",function(req,res){
    res.send("Hello world");

});
app.all("/a*a", function(req,res){
    res.send("hello from a");
})
app.listen(3000);
```
---
### Часто используемые модули. Express
 - Может быть определено несколько обработчиков. Обработчики могут передавать управление следующему в цепочке 
```js 
errorHandler = function(req,res){res.send("error");}
app.get("/error", function(req,res, next) {
    if (1) next();
}, errorHandler);
```
---
### Часто используемые модули. Express
Методы объекта response

|Метод|Описание|
|-----|--------|
|res.download()|	Приглашение загрузки файла.|
|res.end()|	Завершение процесса ответа.|
|res.json()|	Отправка ответа JSON.|
|res.jsonp()|	Отправка ответа JSON с поддержкой JSONP.|
|res.redirect()|	Перенаправление ответа.|
|res.render()|	Вывод шаблона представления.|
|res.send()|	Отправка ответа различных типов.|
|res.sendFile()|	Отправка файла в виде потока октетов.|
|res.sendStatus()|	Установка кода состояния ответа и отправка представления в виде строки в качестве тела ответа.|
---
### Часто используемые модули. Express
 - Работа со статическим содержимым
```js 
app.use(express.static('public'));
```
В данном случае считается, что статическое содержимое находится в директории public, которая находится в корне проекта
---
### Часто используемые модули. Express
 - Автоматический разбор тела (body) и запроса для основных методов HTTP
```js 
const bodyParser =require('body-parser');
app.use(bodyParser.urlencoded(
                                { extended: true }
                            )
        );
app.use(bodyParser.json());
``` 
 - Обращение к параметрам, переданным в запросе: req.params.<param_name>
 - Обращение к параметрам, переданным в теле: req.body. <param_name>
---
### Пример
```js 
const express = require("express");
const app = express();
const bodyParser = require('body-parser');

app.use(bodyParser.urlencoded({extended:true}));
app.use(bodyParser.json());
app.use(express.static('public'));

app.get("/city/:countrycode",function(req,res){
    console.log(req.params.countrycode);
});
app.listen(3000);
``` 
---
### Шаблонизаторы в Express
 - Задача «вставки» динамического содержимого в заранее подготовленный html-документ является типичной (вспоминаем jsp, asp,…)
 - Express поддерживает подключение внешних «движков» шаблонов, обладающих разным синтаксисом и функциональностью.
 - Часто используемые шаблонизаторы:
     - handlebars
     - pug
     - mustache
     - ejs
     - …
---
### Установка шаблонизатора в Express
 - Установка движка
```bash 
npm install --save <имя_движка_в репозитории_npm>
```
 - Регистрация в Express
    - регистрация директории, в которой будут лежать шаблоны:
```js 
app.set('views','./vievs');
``` 
     - регистрация движка
```js 
app.set('view engine','имя движка');
```  
 - Передача параметров движку
```js 
response.render('имя шаблона',<объект с параметрами>);
```  
---
### Шаблонизатор EJS
 - Для подстановки использует выражения javascript, определяемые с помощью тега <% выражение %> (ничего не напоминает?)
 - Примеры выражений:
```js  
<%=title %>
<%=user.name %>
<% for (var i = 0; i < emails.length; i++){ %>
        ....
<%}
%>
```  
---
### EJS. Пример. Сервер
```js  
const express = require('express')
const app = express()

app.set('view engine', 'ejs')

app.use('/', function (request, response) {
    response.render('index', {
        title: 'Мои контакты',
        emailsVisible: true,
        emails: ['test@test.com', 'test@corp.com', 'yuiyuiyuiyuiyiyiuyui'],
        phone: '+1234567890',
    })
})
app.listen(3000)
```        
---
### EJS. Пример. Шаблон
```html  
...
<h1><%=title %> в EJS</h1>
<% if(emailsVisible) { %>
<h3>Адреса электронной почты</h3>
<table>
    <thead>
    <th>#</th> <th>email</th>
    </thead>
    <tbody>
    <% for(let i=0; i<emails.length;i++) { %>
    <tr>
        <td><%=i %></td> <td><%=emails[i] %></td>
    </tr>
    <% } %>
    </tbody>
</table>
<% }else { %>
<h3>Электронный адрес отсутствует</h3>
<% } %>
<p>Телефон: <%=phone %></p>
</body>
<html>
```  
---
### Часто используемые модули. Работа с СУБД
 - Существуют разные типы СУБД
     - Реляционные (Oracle, MSSQLServer, DB2, PostgreSQL, MySQL, …)
     - NoSQL
         - Key-value (Berkeley DB, MemcacheDB, Redis, Riak, Amazon DynamoDB, …)
         - «Документо-ориентированные» (CouchDB, MongoDB, eXist, Berkeley DB XML, …)
     - Иерархические (Caché, …)
     - «Графовые»/сетевые (Neo4j, …)
     - …..
---
### Часто используемые модули. Работа с СУБД
 - Существуют различные модели работы с СУБД (на примере реляционных)
     - Прямой доступ
 ```sql  
 Select * from person
 insert into person() values()
 delete from person where…
 update person set … where …
 ```  
     - При таком доступе работа с данными осуществляется посредством встроенного языка обработки данных СУБД (SQL для реляционных систем)
---
#### Часто используемые модули. Работа с СУБД (ORM-1)
 - Доступ черер ORM-драйвер
```js  
 let Person = db.define('person', {
    name: String,   surname: String,
    age: Number,    male: Boolean,
    data: Object 
 }
 app.use(orm.express("mysql://username:password@host/database", {
    define: function(db, models){
        models.person = Person;
    }
 }))
 );
 Person.find({surname: 'Doe'}, function (err, people){
    //SQL: select * from person where surname = 'Doe'
    console.log(`People found: ${people.length}`);
    console.log(`First person: ${people[0].name}, age ${people[0].age}`);
 });
```  
---
#### Часто используемые модули. Работа с СУБД (ORM-2)
 - Доступ через «ORM» «драйвер»
     - При этом подходе тем или иным образом определяется «модель» (класс сущности) и далее работа в приложении осуществляется с экземплярами классов сущностей, при этом работа по сохранению данных (а также чтению, изменению, удалению) ложится на ORM-систему
     - Каждый из подходов имеет как свои плюсы, так и минусы.
---
### Работа с СУБД (MySQL, без ORM)
 - Установка модуля Node.js для работы с СУБД MySQL:
```bash  
npm install mysql –save
``` 
 - Подключение в проекте:
```js  
const db = require("mysql");
``` 
- Основные методы:
     - createConnection – определение нового подключения к СУБД
     - connection.connect – собственно, подключение
     - connection.query  - выполнение запроса
---
### Работа с СУБД (MySQL, без ORM)
```js  
const db = require("mysql");
const connection = db.createConnection({
    host:"localhost",
    user:"alex",
    password:"Qwerty123",
    database:"world"
});

connection.connect(function(err){
    if (err) {
        console.log(err);
        return;
    }
    console.log("connection established");
})
``` 
---
### Работа с СУБД (MySQL, без ORM)
```js  
connection.query("select * from city", function(err, rows){
    if (err){
        console.log(err);
        return;
    }
    //console.log(rows);
    console.log(JSON.stringify(rows));
});
connection.query("update city set name=?, CountryCode=? where id=?", 
    ["Кстово", "RUS", "100015"],
    function(err, rows){
        if (err){
            console.log(err);
            return;
        }
    }
);
``` 
---
### Пример 
 - Создадим приложение, позволяющее просматривать список стран и городов для них
 - Добавим возможность редактирования параметров города
 - Постараемся выполнить наше приложение в форме Single Page Application, SPA
 - Мы будем использовать AJAX для загрузки нужных данных на клиента
---
### SPA vs MPA
![SPAvsMPA](https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/november/images/dn463786.wasson_figure2_hires(en-us,msdn.10).png)<!-- .element: width="45%"  -->

https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/november/images/dn463786.wasson_figure2_hires(en-us,msdn.10).png<!-- .element: class="copyright-reference"  -->
---
### AJAX
 - AJAX (Asynchronous Javascript and XML) — подход к построению интерактивных пользовательских интерфейсов веб-приложений, заключающийся в «фоновом» обмене данными браузера с веб-сервером. В результате, при обновлении данных веб-страница не перезагружается полностью, и веб-приложения становятся быстрее и удобнее.

https://ru.wikipedia.org/wiki/AJAX <!-- .element: class="copyright-reference"  -->
---
### AJAX
  - SPA, как  подход к построению приложения перекликается с моделью AJAX. При использовании AJAX необходимо иметь специальные имена ресурсов на сервере, при доступе к которым возвращаются необходимые данные <!-- .element: class="left small_font"  -->

![AJAX](https://www.cs.put.poznan.pl/jkobusinski/ajax/model.png)<!-- .element: width="40%"  -->
     
https://www.cs.put.poznan.pl/jkobusinski/ajax/model.png<!-- .element: class="copyright-reference"  -->
---
### AJAX. XMLHttpRequest
Методы
|Метод|	Описание|
|-----|---------|
|abort()|	Отменяет текущий запрос, удаляет все заголовки, ставит текст ответа сервера в null.|
|getAllResponseHeaders()|	Возвращает полный список HTTP-заголовков в виде строки. Заголовки разделяются знаками переноса (CR+LF). Если флаг ошибки равен true, возвращает пустую строку. Если статус 0 или 1, вызывает ошибку INVALID_STATE_ERR.|
|getResponseHeader(headerName)|	Возвращает значение указанного заголовка.Если флаг ошибки равен true, возвращает null.Если заголовок не найден, возвращает null. Если статус 0 или 1, вызывает ошибку INVALID_STATE_ERR.|
|open(method, URL, async, userName, password)|	Определяет метод, URL и другие опциональные параметры запроса; параметр async определяет, происходит ли работа в асинхронном режиме. Последние два параметра необязательны.|
|send(content)|	Отправляет запрос на сервер.|
|setRequestHeader(label, value)|	Добавляет HTTP-заголовок к запросу.|

Свойства
|Свойство|	Тип	|Описание|
|--------|------|--------|
|onreadystatechange|	EventListener|	Обработчик события, которое происходит при каждой смене состояния объекта. Имя должно быть записано в нижнем регистре.|
|readyState	|unsigned short|	Текущее состояние объекта (0 — не инициализирован, 1 — открыт, 2 — отправка данных, 3 — получение данных и 4 — данные загружены)|
|responseText|	DOMString|	Текст ответа на запрос. Если состояние не 3 или 4, возвращает пустую строку.|
|responseXML|	Document|	Текст ответа на запрос в виде XML, который затем может быть обработан посредством DOM. Если состояние не 4, возвращает null.|
|status|	unsigned short|	HTTP-статус в виде числа (404 — «Not Found», 200 — «OK» и т. д.)|
|statusText|	DOMString|	Статус в виде строки («Not Found», «OK» и т. д.). Если статус не распознан, браузер пользователя должен вызвать ошибку INVALID_STATE_ERR.
---
### AJAX. Пример
```js  
var oReq = new XMLHttpRequest();
var request= “http://localhost:3000/country”; 
console.log(request);
oReq.addEventListener("load", reqListener);
oReq.open("GET", request);
oReq.send();

function reqListener(event) {
    var data = JSON.parse(this.responseText);
   …
}
``` 
---
### AJAX. fetch
 - Использование fetch (есть поддержка во всех современных браузерах):
     - fetch(url, [options]) - Выполняет запрос на ресурс url (по умолчанию GET), с использованием (необязательно) опций. Возвращает Promise c объектом response, содержащий ответ сервера.
Объект Responce:
|Метод/Свойство|Описание|
|--------------|--------|
|responce.status|Код возврата РЕЕЗ-запроса (200 при удачном завершении)|
|response.text()| читает ответ и возвращает как обычный текст|
|response.json()| декодирует ответ в формате JSON|
|response.formData()| возвращает ответ как объект FormData|
|response.blob()| возвращает объект как Blob (бинарные данные с типом)|
|response.arrayBuffer()| возвращает ответ как ArrayBuffer (низкоуровневое представление бинарных данных)|
|response.body|  возвращает объект ReadableStream, с помощью которого можно считывать тело запроса по частям|
---
### AJAX. Использование fetch
  - При необходимости выполнить запрос с другим методом протокола HTTP (например, POST) используются дополнительные параметры:
  
```js
let country = {id: 1, name: 'Москва'};
fetch('/country', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json;charset=utf-8'
    },
    body: JSON.stringify(ccountry)
}
)
    .then(responce => responce.json())
    .then(json => console.log(json));
```

---
### AJAX
 - Существует множество библиотек, которые предоставляют более удобное API, чем XMLHttpRequest (jQuery, Prototype, …)
 - Есть ограничения безопасности при выполнении AJAX – запросов
     - Нельзя выполнять запросы из страниц, загруженных локально (с жесткого диска)
     - Особенности при обработке файлов cookes
---
### REST API
 - REST (Representational State Transfer) — архитектурный стиль взаимодействия компонентов распределённого приложения в сети. 
 - Основные идеи:
     - Каждый «объект» однозначно описывается своим url (http://localhost/country/Russia или http://localhost/country/7 )
     - Для манипуляций с объектом достаточно CRUD операций
     - Все CRUD операции можно выразить методами протокола HTTP
---
### REST API
|HTTP Meth	|CRUD	|Entire Collection (e.g. /customers)	|Specific Item (e.g. /customers/{id})|
|-----------|-------|---------------------------------------|------------------------------------|
|POST	|Create	|201 (Created), 'Location' header with link to /customers/{id} containing new ID.	|404 (Not Found), 409 (Conflict) if resource already exists..|
|GET	|Read	|200 (OK), list of customers. Use pagination, sorting and filtering to navigate big lists.	|200 (OK), single customer. 404 (Not Found), if ID not found or invalid.|
|PUT	|Update/Replace	|405 (Method Not Allowed), unless you want to update/replace every resource in the entire collection.	|200 (OK) or 204 (No Content). 404 (Not Found), if ID not found or invalid.|
|PATCH	|Update/Modify	|405 (Method Not Allowed), unless you want to modify the collection itself.	|200 (OK) or 204 (No Content). 404 (Not Found), if ID not found or invalid.|
|DELETE	|Delete	|405 (Method Not Allowed), unless you want to delete the whole collection—not often desirable.	|200 (OK). 404 (Not Found), if ID not found or invalid.|
---
### REST API. Пример (сервер)
```js
const express = require("express");
const app = express();
const db = require("mysql");
const connection = db.createConnection({
    host:"localhost",   user:"alex",
    password:"Qwerty123",    database:"world"
});
connection.connect(function(err){
    if (err) { console.log(err); return;}
    console.log("connection established");
});
app.get("/country",function(req,res){
connection.query("select * from country", 
        function(err, rows){ 
            if (err){console.log(err);return;}
            res.end(JSON.stringify(rows));
        });
}); 
app.listen(3000);
```
---
### REST API. Пример (клиент)
```js [1-25]
function queryHandbook(request, rowHandler) {
    function reqListener(event) {
        var data = JSON.parse(this.responseText);
        var table = document.getElementById("table_data");
        table.innerHTML=‘’;
        if (data.length > 0) {
            var header = table.createTHead();   var hrow = header.insertRow();
            for (var k in data[0]) {
                var cell = hrow.insertCell();   cell.innerHTML= k;
            }
        }
        for (var i =0; i<data.length;i++ ){
            var newRow = table.insertRow();
            for (j in data[i]){
                var cell = newRow.insertCell();  cell.innerHTML = data[i][j];
            }
        }
    }
    var oReq = new XMLHttpRequest();
    oReq.addEventListener("load", reqListener);
    oReq.open("GET", request);    oReq.send();
}
```
---
### REST API. Пример (сервер)
 - Развиваем сервер, добавляем дополнительные операцию получения списка городов по коду страны

```js 
const bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({ extended: true }));
…
app.get("/city/:countrycode",function(req,res){
    console.log(req.params.countrycode);
    connection.query("select * from city where countryCode=?", req.params.countrycode, 
            function(err, rows){
                if (err){  console.log(err);  return;}
                res.end(JSON.stringify(rows));
            });
});
```
---
### REST API. Пример (клиент)
 - Развиваем клиента, добавляем запрос на города при нажатии на строку со страной

```js 
let countryHandler = function(){
  queryHandbook('http://localhost:3000/city/'+ 
                    this.cells[0].innerHTML, cityHandler);
};

//…

newRow.onclick = countryHandler;
```
---
### REST API. Пример (сервер)
 - Развиваем сервер, добавляем операцию редактирования сведений о городе

```js 
var bodyParser = require('body-parser');

app.use(bodyParser.json());

app.put("/city", function(req,res){
    console.log("PUT /city called");
    console.log(req.body);
    connection.query("update city set name=?, CountryCode=?,"
                            +" District=?, Population=? where id=?", 
                    [req.body.Name, req.body.CountryCode, 
                        req.body.District, req.body.Population, req.body.ID], 
                        function(err, rows){
                            if (err){
                                console.log(err);
                                return;
                            }
                            //console.log(rows);
                            //res.end(JSON.stringify(rows));
                        });
});
```
---
### REST API. Пример (клиент)
 - Развиваем клиента, добавляем редактируемые поля в таблицу и запрос к REST-сервису

```js 
function editCityHandler(){
    var edits =  document.getElementsByName("dynamic_form");
    var city = new Object();
    var table = document.getElementById("table_data");
    var colls = table.tHead.rows[0].cells;
    for (var i = 0; i< edits.length;i++){
        city[colls[i].innerHTML] = edits[i].value;
        console.log();
    }
    console.log(city);

    var oReq = new XMLHttpRequest();
    //oReq.addEventListener("load", reqListener);
    oReq.open("PUT", "http://localhost:3000/city/");
    oReq.setRequestHeader('Content-type', 'application/json; charset=utf-8');
    oReq.send(JSON.stringify(city));
    console.log("sended");
}
```
---
### Сессии
 - Протокол HTTP не поддерживает состояние (Stateless protocol)
 - Это означает, что 
     - Не сохраняет состояния между вызовами
     - Все взаимодействие имеет короткий жизненный цикл (запрос - ответ)
     - КАЖДЫЙ ресурс, доступ к которому осуществляется через HTTP, получается отдельным запросом, без какой-либо связи с предыдущими запросами (в HTTP 1.1 добавлен механизм постоянного соединения)
---
### Сессии
 - Часто приложениям необходим механизм поддержки состояния приложения (соединения пользователя)
     - Пользователь уже заходил на этот сайт
     - Пользователь аутентифицировался
     - Пользователь сохранил свои настройки
     - Пользователь сохранил данные (корзину при покупке в магазине)
     - …
 - => Такой механизм должен реализовываться поверх протокола HTTP
---
### Сессии
 - Как реализовать состояние?
 - Как клиенту однозначно идентифицировать себя на сервере?
 - Как серверу предоставить конкретный контент для каждого клиента?
---
### Добавление состояния в HTTP (версии 1.0 и младше)
 - Существуют различные способы поддержания состояний с использованием HTTP (не являющиеся частью протокола). Например,
 - Клиентские механизмы:
     - Куки (Cookies)
     - Скрытые переменные
     - Локальное хранилище
 - Серверные механизмы:
     - сеансы
---
### Куки
 - Небольшой объем информации, отправляемой сервером клиенту (браузеру), который сохраняется на клиенте и затем отправляется обратно клиентом при запросах
 - Используется для:
     - проверки подлинности
     - отслеживания пользователей
     - сохранение пользовательских настроек, корзин и т.д.
---
### Куки
 - Формируется сервером
 - Отправляется клиенту с заголовком HTTP-ответа
 - Возвращается браузером серверу в HTTP-запросе

![Cookies](https://upload.wikimedia.org/wikipedia/commons/b/bc/HTTP_cookie_exchange.svg)
https://upload.wikimedia.org/wikipedia/commons/b/bc/HTTP_cookie_exchange.svg<!-- .element: class="copyright-reference"  -->

---
### Куки. HTTP запросы
```txt
GET /index.html HTTP/1.1                браузер -> сервер (запрос)
Host: www.example.org
________________________

HTTP/1.1 200 OK                         сервер -> браузер (ответ)
Content-type: text/html
Set-Cookie: name=value                  установка куки

(содержимое страницы)

________________________

GET /spec.html HTTP/1.1                 браузер -> сервер (запрос)
Host: www.example.org
Cookie: name=value                      все последующие запросы содержат 
Accept: */*                                 установленную куки

```
---
### Поддержка куки (node.js)
```js
const http = require("http");
http.createServer(function(req,res){
    res.writeHead(200, {
        "Set-Cookie":"testcookie=test",
        "Context-type":"text/plain"
    })
    res.end("Hello world");
}).listen(3000);
```
---
### Куки
![CookieBrowser](../img/Cookie.png)
---
### Доступ к куки на стороне клиента
 - Из JavaScript (браузер) доступно поле document.cookie
 - Поле можно устанавливать/читать вручную
```js
document.cookie = "username=ivan;password=12345";
```
 - Или с использованием функции
```js
Cookies.set("username", “ivan");
...
alert(Cookies.get("username")); // ivan
```
---
### Время жизни куки
 - session cookie - тип по умолчанию 
     - временный файл, который хранится только в памяти браузера
     - когда браузер закрыт, временные файлы куки уничтожаются
     - не может использоваться для сохранения долгосрочной информации
     - безопаснее, поскольку никакие другие программы, кроме браузера не могут получить к ним доступ
---
### Время жизни куки
 - persistent cookie: куки, которые хранятся в файловой системе компьютера
     - может сохранять информацию между перезагрузками браузера
     - потенциально менее безопасный, поскольку пользователи (или программы) могут открывать файлы куки, видеть/изменять значения файлов куки и т. д.
     - Для куки может быть определено время устаревания:

```js
document.cookie="username=Ivan Drago; expires=Thu, 17 Jul 2017 15:00:00 GMT";
```
---
### Сессии на сервере
 - Текущее состояние сохраняется на сервере (в файле, база данных, в памяти)
 - Каждый запрос включает в себя токен, идентифицирующий сессию браузера (токены могут быть переданы через куки, скрытые переменные, переписывание URL-адресов).
 - При каждом запросе исполняющий скрипт использует токен для получения состояния сеанса
---
### Сессии в Node.js
 - Имеется большое количество модулей, упрощающих работу с сессиями
 - При использовании express можно использовать модуль express-session
 - Сессии можно хранить либо в памяти сервера
     - Занимают оперативную память
     - Теряются при перезапуске сервера
 - Либо в базе данных
 - При использовании СУБД MySQL для хранения сессий можно использовать модуль express-mysql-session
---
### Сессии. пример
```js
const express = require("express");
const bodyParser = require('body-parser');
const cookieParser = require('cookie-parser');
const session = require('express-session');
const SessionStore = require('express-mysql-session');

const app = express();

app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
app.use(express.static('public'));
app.use(cookieParser());
```
---
### Сессии. пример
 - Модуль express-session использует куки для формирования токенов сессии. 
 - Для обеспечения удобного механизма работы с куки подключаем модуль  cookie-parser
 - Инициализируем наше приложение, передавая ему cookie-parser
---
### Сессии. пример
```js
const options = {
    host: 'localhost',
    user: 'alex',
    password: 'Qwerty123',
    database: 'world'
};


app.use(session({
    key: 'session_cookie_name',
    secret: 'session_cookie_secret',
    store: new SessionStore(options),
    resave: true,
    saveUninitialized: true
}))
```
---
### Сессии в Node.js
 - Инициализируем хранилище сессий, используя в качестве СУБД MySQL
 - Параметры подключения указываем в options
     - Secret – ключ, используемый для подписи файла куки
     - Store – используемое хранилище
     - Resave – принудительная запись сеанса в хранилище, даже если он не изменялся
     - saveUninitialized – в хранилище записываются в том числе неинициализированные сессии
---
### Сессии. пример
```js
app.get('/', function(req, res, next) {
    req.session.number = req.session.number + 1 || 1;
    console.log(req.session.number)
    res.end("You read this page "+req.session.number.toString()+" times");
    next();
})

app.listen(3000);
```
Сессия - обычный объект, в который можно добавлять свойства
---
### Практическое использование сессий
 - Разработаем систему, позволяющую закрыть доступ к «секретной» информации всем «неавторизованным» пользователям
 - Система при запросе секретных данных должна позволить пользователю авторизоваться (ввести логин/пароль), а в случае, если он уже авторизовался – предоставить данные
 - Сессия пользователя (авторизация) должна сохраняться при запросе к другим документам
---
### Пример. Подключение модулей
```js
const express = require("express");
const bodyParser = require('body-parser’);
const cookieParser = require('cookie-parser’);
const session = require('express-session’);
const SessionStore = require('express-mysql-session’);
const connection = require("./db");
const app = express();

app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
app.use(express.static('public'));
app.use(cookieParser());
```
---
### Пример. Настройка сессий
```js
const options = {
    host: 'localhost',
    user: 'alex',
    password: 'Qwerty123',
    database: 'world'
};
app.use(session({
    key: 'session_cookie_name',
    secret: 'session_cookie_secret',
    store: new SessionStore(options),
    resave: true,
    saveUninitialized: true
}))
```
---
### Пример. Доступ к данным
```js
app.get('/topsecret/:document', checkUser, function(req, res) {
    res.end("Вы получили доступ к секретным данным "+ req.params.document);
});

app.listen(3000);
```
---
### Пример. Проверка пользователя
```js
function checkUser(req, res, next) {
    if (req.session.user_id) {
        connection.query("select count(*) cnt from users where user_name=?"
                                                            +" and passwd=?", 
            [req.session.user_id,req.session.password], function(err, rows){
            if (err){
                console.log(err);
                return;
            }
            if (rows[0].cnt == 1) next();
            else {
                req.session.url = req.url;
                res.redirect('/login.html');
            }
        });
    } else {
        req.session.url = req.url;
        res.redirect('/login.html');
    }
}
```
---
### Пример. Операции с сессиями
```js
app.post('/session/create', function(req, res) {
    req.session.user_id = req.body.login;
    req.session.password = req.body.password;

    if (req.session.url)
        res.redirect(req.session.url);
});

app.get('/session/del', function(req, res) {
    if (req.session) {
        req.session.destroy(function() {});
    }
    res.end("Session deleted");
});
```
---
### Пример. Login.html
```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
</head>
<body>
<form action="/session/create" method="POST">
    <fieldset class="fieldset-auto-width">
        <legend>Вход в систему</legend>
        <label for="login">Имя пользователя</label>
        <input type="text" name="login">
        <label for="password">Пароль</label>
        <input type="text" name="password">
        <input type="submit">
    </fieldset>
</form>
</body>
</html>
```
---
### Взаимодействие клиента и сервера
 - Протокол HTTP предполагает сценарий взаимодействия, при котором инициатором акта обмена данными выступает клиент (клиент посылает запрос серверу)
 - Часто встречаемая ситуация –сервер должен отправить какие-то данные клиенту (должен выступить инициатором взаимодействия)
     - Обмен сообщениями между клиентами
     - Игры
     - Оповещения о событиях
     - …
---
### Варианты реализации
 - Периодические запросы к серверу (не появились ли у сервера новые данные)
 - LongPooling
 - WebSocket
 - HTML5 Server Sent Events(SSE)
 - …
 - Многочисленные библиотеки, в реальности использующие один из описанных выше механизмов
---
### Long pooling
 - Клиент запрашивает страницу у сервера, используя обычный http
 - На странице (обычно) выполняется JavaScript, который запрашивает ресурс у сервера.
 - Сервер НЕ реагирует на запрос (не формирует ответ и не закрывает соединение) и ждет, пока не появится необходимость в пересылке сообщения клиенту
 - Когда появляется новая информация, сервер отсылает ее клиенту
 - Клиент получает новую информацию и НЕМЕДЛЕННО отсылает другой запрос серверу, запуская процесс ожидания на нем снова.
---
### Long pooling
![LongPooling](https://javascript.info/article/long-polling/long-polling.svg)<!-- .element: class="big_image"  -->

https://javascript.info/article/long-polling/long-polling.svg<!-- .element: class="copyright-reference"  -->
---
### Long pooling. Шаблон клиента
```js
<script>
//    …
    request();
    function request() {
        var req = new XMLHttpRequest();
        req.open("GET", "/messages", true);
        req.onload = function () {
            //обрабатываем событие сервера
            request();//снова запрос 
        }
        req.send('');
    }
</script>
```
---
### Long pooling. Сервер
```js
let clients = [];

app.post('/publish', function(req, res) {
    console.log("get request for publlish "+ req.body.message);
    //console.log(req);
    for (let i=0; i<clients.length;i++){
        clients[i].end(req.body.message);
    }
    clients = [];

})

app.get('/messages', function(req, res) {
    console.log("get request for messages...");
    clients.push(res);
})
```
---
### Server-sent events (часть HTML5)
 - Клиент запрашивает ресурс у сервера, используя обычный http
 - Сервер отдает поток данных (для этого он отправляет клиенту специальные заголовки, показывающие, как он должен обработать этот поток), не закрывая соединения
 - Клиент использует специальный объект (EventSource) для того, чтобы обработать поток данных с сервера
 - Поток данных структурирован на сообщения, у каждого из которых может быть уникальный идентификатор и тип и должна быть содержательная часть (данные)
---
### SSE. Шаблон клиента
```html
<div id="chat">
    <h1 id="message">0</h1>
    <button onclick="send()">Start</button>
</div>
<ul id="messages"></ul>
<script>
    function send(){
        console.log("begin script");
        let eventSource = new EventSource("/sse")
        eventSource.addEventListener("message", (e)=>{
            try {
                console.log(e.data);
                message.innerHTML = Math.round(e.data);
            }catch{
                console.log("error");
            }
        })
    }
</script>
```
---
### SSE. Шаблон сервера
```js
app.get("/sse", (req, res) => {
    res.set("Content-Type","text/event-stream")
    res.set("Connection","keep-alive");
    res.set("Cache-Control","no-cache");
    res.set("Access-Control-Allow-Origin","*");
    console.log("client connected");

    let id = 0;
    let timer = setInterval(() => {
        res.status(200);
        let value = Math.random() * 100;
        let data = 'id:' + (id++) + '\n' + 'data: ' + value + '\n\n';
        res.write(data); // НЕ send!!!
        console.log("data send: " + data);
    }, 1000);

    req.on('close', () =>{
        console.log("close connection" );
        clearInterval(timer);
    })
})
```
---
### «Чат» с использованием SSE 
 - Клиент создает соединение и читает поток данных (сообщений) с сервера
 - Новое введенное сообщение отправляет серверу (используется метод POST)
 - Сервер для клиентского соединения создает поток событий и регистрирует обработчик, который запустится при наступлении определенного события (прихода нового сообщения)
 - В обработчике метода POST данные переданные клиентом упаковываются в событие и передаются соответствующему обработчику (см. выше)
---
### SSE. Чат. Клиент
```html
<form id="chat">  <input type="text" id="message"> <input type="submit"> </form>
<ul id="messages"></ul>
<script>
    listen();
    function listen(){
        let eventSource = new EventSource("/messages")
        eventSource.addEventListener("message", (e)=>{
            try {
                var el = document.createElement("li");
                el.textContent = e.data; messages.appendChild(el);
            }catch{console.log("error");}
        })
    }
    chat.onsubmit=function(){
        var req = new XMLHttpRequest();
        req.open("POST","/publish",true);
        req.setRequestHeader('Content-type', 'application/json; charset=utf-8');
        req.send(JSON.stringify({message: this.elements.message.value}));
        return false;
    }
</script>
```
---
### SSE. Чат. Сервер (1)
```js
const express = require("express");
const app = express();
const bodyParser = require('body-parser');

app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
app.use(express.static('public'));

const EE = require("events").EventEmitter;
const messageEmitter = new EE();

app.post("/publish", (req, res) =>{
    console.log("get request for publlish "+ req.body.message);
    messageEmitter.emit('newmessage',{data: req.body.message});
    res.end();
})
```
---
### SSE. Чат. Сервер (2)
```js
app.get("/messages", (req, res) => {
    res.set("Content-Type","text/event-stream")
    res.set("Connection","keep-alive");
    res.set("Cache-Control","no-cache");
    res.set("Access-Control-Allow-Origin","*");
    console.log("client conected");
    let id = 0;
    const callback = function(message){
        res.status(200);
        let data = 'id:'+(id++)+'\n'+'data: '+message.data+'\n\n';
        res.write(data); // НЕ send!!!
        console.log("data send: "+data);
    }
    //устанавливаем обработчик для вновь пришедшего сообщения
    messageEmitter.on('newmessage', callback);/
    req.on('close', () =>{
        console.log("close connection" );
        //при закрытии соединения - убираем обработчик
        messageEmitter.off('newmessage', callback);
    })
})
app.listen(3000);
```
---
### WebSockets (Википедия)
 - WebSocket — протокол **полнодуплексной** связи (может передавать и принимать одновременно) поверх TCP-соединения, предназначенный для обмена сообщениями между браузером и веб-сервером в режиме реального времени.
 - Протокол WebSocket — это независимый протокол, основанный на протоколе TCP. Он делает возможным более тесное взаимодействие между браузером и севрером, способствуя созданию приложений реального времени.
     - В настоящее время в W3C осуществляется стандартизация API Web Sockets. Черновой вариант стандарта этого протокола утверждён IETF.
    - _Широко поддерживается современными браузерами и серверами_
---
### WebSocket. Клиент
 - Создание
```js
 socket=new WebSocket("ws://localhost:8080/ws");
```
 - Обработчики событий
     - Соединение 
```js
socket.onopen=function(){…}
```
     - Отсоединение 
```js
socket.onclose = function(event) {
    if (event.wasClean) {
        console.log('Соед. закрыто чисто');
    } else {
        console.log('Обрыв соединения'); 
    }
    console.log('Код: ' + event.code + ' причина: ' + event.reason);
};
```
---
### WebSocket. Клиент
 - Прием сообщения
```js
socket.onmessage = function(event) {
    console.log("Получены данные " + event.data);
    … = event.data;
};
```
 - Ошибка 
```js
socket.onerror = function(error) {
    console.log("Ошибка " + error.message);
}
```
---
### WebSocket. Сервер (1)
 - Модуль ws

```js
const express = require('express');
const http = require('http');
const url = require('url');
const WebSocket = require('ws');
const bodyParser = require('body-parser');

const app = express();
app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());
app.use(express.static('public'));

const EE = require("events").EventEmitter;
const messageEmitter = new EE();

var server = http.createServer(app);
var wss = new WebSocket.Server({ server });
let count  = 0;

server.listen(8080, function listening() {
    console.log('Listening on %d', server.address().port);
});

```
---
### WebSocket. Сервер (2)
```js
wss.on('connection', function connection(ws, req) {
    var location = url.parse(req.url, true);
    console.log("new client connected");
    let instance = count++;
    const callback = (message) => {
        console.log("in callback: message="+message.data);
        ws.send(message.data);
        console.log("sending to client....%s", instance);
    }
    messageEmitter.on('newmessage',callback);

    ws.on('message', function incoming(message) {
        console.log('received: %s', message);
        messageEmitter.emit('newmessage',{data: JSON.parse(message).message});
    });
    ws.on('close', function close() {
        console.log('socket closed for client %s', instance);
        messageEmitter.off('newmessage',callback);
    })
});
app.listen(3000);

```
---
### WebSocket. Клиент
```html
<form id="chat"> <input type="text" id="message"> <input type="submit"> </form>
<ul id="messages"></ul>
<script>
    var socket = new WebSocket("ws://localhost:8080/ws");
    socket.onopen = function() {console.log("Соединение установлено."); };
    socket.onclose = function(event) {
        if (event.wasClean) {console.log('Соединение закрыто чисто');
        } else {console.log('Обрыв соединения');}
        console.log('Код: ' + event.code + ' причина: ' + event.reason);
    };
    socket.onmessage = function(event) {
        console.log("Получены данные " + event.data);
        var el = document.createElement("li");
        el.textContent = event.data;
        messages.appendChild(el);
    };
    socket.onerror = function(error) {console.log("Ошибка " + error.message);}
    chat.onsubmit=function(){
        socket.send(JSON.stringify({message: this.elements.message.value}));
        return false;
    }
</script>
```