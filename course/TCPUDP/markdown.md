# План занятия
- Использование API java.net для работы с
    - UDP
    - TCP
- Примеры приложений (разработка, компиляция, запуск), использующих эти протокол

---
## Примитивы передачи данных
- Распределенные системы нуждаются в обмене данными и синхронизации между автономными распределенными процессами
    - Interprocess communication (IPC)
        - Разделяемые переменные
        - Передача сообщений
- Синхронизация
    - Процессы на различных компьютерах выполняются с различными скоростями
- Примитивы передачи данных
    - send expression_list to destination
    - receive variable_list from source
- Вопросы
    - Как определять адреса (как указывать процесс, которому передаются данные)?
    - Как добиваться синхронизации при передаче

---
## Передача сообщений. Примитивы 
- Каналы (Pipes). Локальный IPC.
    - Давний и один из самых простейших средств IPC 
    - Позволяют двум процессам взаимодействовать, используя буфер, реализуемый ядром ОС; данные сохраняются в буфере в порядке очереди (FIFO) 
    - Каналы обеспечивают возможность двум «родственным» (например, родитель/потомок) процессам взаимодействовать
- Другие локальные IPC
    - Очереди
    - Именованные каналы
    - Семафоры
    - Разделяемые переменные
    - …


---
## Передача сообщений. Примитивы 
Сокеты (Socket`s). Распределенный IPC.<!-- .element: class="left" -->
- Сокеты предоставляют мощнейший механизм взаимодействия распределенных процессов, однако при их использовании требуется аккуратность (на программиста возлагается ответственность за детали взаимодействия)
- Существуют стандартные интерфейсы, механизм поддерживается на всех значимых платформах
- Сокет предоставляет механизм для взаимодействия двух процессов
- Сокеты существуют, пока существуют взаимодействующие процесс


---
## Примитивы передачи данных
Указание назначения (адресата)
- Прямое наименование: имена процессов получателя и отправителя используются в качестве адресатов (пара имен однозначно определяет канал)

        - send cur_status to monitor 
        - receive message from handler
    - Просто реализовать и использовать
    - Позволяет процессу легко контролировать когда получать какое сообщение с какого процесса
    - Используется для реализации клиент-серверных приложений
        - хорошо подходящий способ, чтобы реализовать схему клиент/сервер, если есть один клиент и один сервер
        - в противном случае, сервер должен уметь принимать запросы от любого клиента в любое время,и клиент должен уметь обращаться к множеству сервисов в одно время, если доступно больше одного сервера

---
## Примитивы передачи данных
Указание назначения (адресата)
- глобальные имена или почтовые ящики: независимое имя процесса-приемника может использоваться процессами-источниками
    - Сообщения, посланные в почтовый ящик, могут получаться любым процессом
    - Чтобы реализовать  концепцию клиент-сервера
        - клиенты посылают сообщения в почтовый ящик, освободившийся сервер их обрабатывает
    - недостаток: дорогая реализация
        - сообщение послано в ящик
        - если один процесс решил получить сообщение, он должен его заблокировать
        - взаимное исключение при параллельном доступе
- Порты: почтовый ящик, но только одному процессу разрешаются получать сообщения
    - легко осуществимо - принимать может только один процесс - не нужна блокировка
    - подходит если один сервер и много клиентов
- Указание адресов при программировании в Internet
    - гибридная прямая схема присваивания имен/порта
    - порты соответствуют концепции “много процессов посылают, один принимает”

---
## Примитивы передачи сообщений
Семантика примитивов передачи сообщений
- Блокировка
    - не блокирующие: вызов не задерживает вызывающий процесс (управление возвращается сразу)
    - блокирующий: вызов не возвращает управление до завершения
- Синхронизация
    - синхронные: нет никакой буферизации
        - процессы синхронизируются по любому сообщению
        - источник блокируется до тех пор, пока получатель не будет готов к приему
        - приемщик блокируется, пока процесс-источник не будет готов к передаче
    - асинхронные: сообщения передаются с использованием неограниченного буфера
        - передающий процесс выполняет передачу неограниченное число раз
        - передающий никогда не блокируется
        - принимающий блокируется на пустой очереди
- буферизованные: буферизация с ограниченным буфером
    - передающий процесс может передавать до тех пор, пока не переполнился буфер
    - передающий блокируется при переполненном буфере
    - принимающий блокируется на пустой очереди


---
## Примитивы передачи сообщений
Не блокирующие примитивы для асинхронной или буферизированной передачи
- прием
    - фоновый вариант: процесс продолжает выполняться, при приеме получает прерывание (например, callback)
        - иногда сложно для реализации
    - программные опросы 
- отправка
    - передающий процесс ожидает освобождения буфера или удаляет из него не посланные сообщения

---
## Сокеты. TCP

---
## Механизм сокетов
- IPC основанный на TCP
    - Абстрактный сервис: поток байт принимается и получается
    - возможности
        - Размер сообщения: нет ограничений, TCP решает, когда послать сообщение транспортного уровня, состоящее из нескольких прикладных сообщений, непосредственная (немедленная) передача может ,быть принудительной
        - Ориентированный на соединение
        - Основанный на таймаутах механизм отслеживания потерянных сообщений
        - Очередь на приемщике
        - Блокирование при приеме
        - Отслеживание переполнения буферов
        - Сервер должен создавать новый поток для каждого принятого соединения
- API для потоков
    - Создание соединения
        - клиент: запрос коннекта
        - сервер: слушает порт и принимает запросы на соединение. Принимает соединение. Создает новый поток для соединения
---
## Сокеты. UDP

---
## Механизм сокетов
- IPC базирующийся на UDP
    - Свойства UDP: нет гарантии порядка сообщений, сообщения теряются и дублируются
    - Необходимые шаги
        - создание сокета
        - связывание сокета с портом
            - клиент: произвольный свободный порт
            - сервер: порт сервера
    - Метод приемки: возвращает [Интернет адрес и порт отправителя] + [сообщение]
    - Размер сообщения: IP разрешает сообщения до 216 = 65536 байт
        - Большинство реализаций ограничивают 8 Kb
        - Большие сообщения увеличивают производительность передачи
        - Если передаваемое сообщение слишком велико, оно усекается
    - Отправка – не блокирующая 
    - Прием сообщений блокирующий
---
## Прикладные протоколы
- Типы сообщений (пример)
- С –клиент, S –сервер

| Код | Тип | Отправитель | Получатель | Комментарий |
| :-- | :--: | ---------: | ---------: | ----------: |
|1  |(REQ)	|request    		|C             |S             	|Запрос клиента на получение данных|
|2  |(REP)  	|reply                  |S             |C             	|Ответ сервера на запрос клиента   |
|3  |(ACK)  	|ack                    |S/C           |C/S          	|Предыдущее сообщение доставлено   |
|4  |(AYA)  	|are you alive?       	|C             |S              	|Тестовое сообщение для проверки работоспособности сервера |
|5  |(IAA)   	|I am alive             |S             |C             	|Ответ сервера о его работоспособности |
|6  |(TA)    	|try again              |S             |C             	|Сервер перегружен и не имеет ресурсов для обработки запроса |

---
## Прикладные протоколы
- Основными типами сообщений являются типы 1 и 2 
- Тип 3 служит для повышения надежности 
- Типы 4-6: не обязательны, но добавляют дополнительную функциональность
- Необходимость в типах 4-5: Предположим, что клиент послал запрос. Что, если нет ответа, за приемлемое время? Сервер все еще работает или сервер потерпел крах?
    - Клиент использует AYA сообщение для проверки сервера, если IAA (или REP) сообщение получено, значит все в порядке;  Иначе, если после нескольких  AYA сообщений, нет  обратных IAA/REP, клиент может предположить, что сервер недоступен
---
## Примеры

---
## Java API для UDP (java.net)
- Класс DatagramPacket 
    - Представляет собой пакет для передачи по сети. Обычно содержит адрес и порт процесса-получателя
- Класс DatagramSocket 
    - Служит для получения и отправки пакетов данных посредством UDP
    - send (DatagramPacket dp) отправляет пакет
    - receive(DatagramPacket p) принимает пакет
---
## Пример использования UDP (java.net)
- Сервер
    - Создает DatagramSocket и связывает его с определенным портом
    - Ожидает сообщение от клиента 
    - Формирует пакет – ответ, перенося в него данные, полученные от клиента, адрес и порт клиента извлекает из пришедшего пакета
    - Отправляет пакет-ответ клиенту
- Клиент
    - Создает DatagramSocket
    - Создает DatagramPacket
    - Упаковывает в пакет необходимые данные и параметры (адрес и порт сервера)
    - Передает пакет на сервер
    - Ожидает сообщение от сервера
    - Печатает пришедшее сообщение
---
## Клиент (UDP, java.net)
```java
import java.net.*;
import java.io.*;
public class UDPClient{
    public static void main(String args[]){ 
 // args give message contents and destination hostname
 try {
  DatagramSocket aSocket = new DatagramSocket();      // create socket 
  byte [] message = args[0].getBytes();
  InetAddress aHost = InetAddress.getByName(args[1]); // DNS lookup
  int serverPort = 8080;                                                   
  DatagramPacket request = 
    new DatagramPacket(message,  args[0].length(), aHost, serverPort);
  aSocket.send(request);     //send message
  byte[] buffer = new byte[1000];
  DatagramPacket reply = new DatagramPacket(buffer, buffer.length); 
  aSocket.receive(reply);     //wait for reply
  System.out.println("Reply: " + new String(reply.getData())); 
  aSocket.close();
 } catch (SocketException e){ System.out.println("Socket: " + e.getMessage()); 
  // socket creation failed
 } catch (IOException e){ System.out.println("IO: " + e.getMessage()); 
  // can be caused by send
 }
    }
}
```

---
## Сервер (UDP, java.net)
```java
import java.net.*;
import java.io.*;
public class UDPServer {
    public static void main(String args[]) {
        try (DatagramSocket aSocket = new DatagramSocket(8080)) {
            // create socket at agreed port
            byte[] buffer = new byte[1000];
            while (true) {
                DatagramPacket request = new DatagramPacket(buffer, buffer.length);
                aSocket.receive(request);
                DatagramPacket reply = new DatagramPacket(request.getData(), request.getLength(),
                        request.getAddress(), request.getPort());
                aSocket.send(reply);
            }
        } catch (SocketException e) {
            System.out.println("Socket: " + e.getMessage()); // socket creation failed
        } catch (IOException e) {
            System.out.println("IO: " + e.getMessage());
        }
    }
}
```
---
## Сервер (UDP, java.net)
- ServerSocket представляет сокет на стороне сервера
    - В конструкторе принимает порт, на котором будут ожидаться соединения клиентов
    - Для ожидания клиентов вызывает блокирующий метод accept, возвращающий Socket
- Socket класс для работы с соединением (клиент и сервер)
    - Конструктор для создания сокета и соединения с удаленным узлом и портом 
    - Методы для работы с входными и выходными потоками

---
## Пример использования TCP (java.net)
- Сервер
    - Создает ServerSocket, связывает его с портом, на котором будут ожидаться клиенты
    - Ожидает клиентов (accept)
    - При подключении клиента, создает новый класс-Thread (Connection), передавая ему клиента для обработки
    - Класс Connection создает потоки ввода и вывода, ассоциированные с сокетом
    - Считывает данные клиента, а затем передает их же клиенту, после чего прекращает свою работу
- Клиент
    - Создает Socket, соединяется с сервером
    - Создает потоки ввода и вывода, связанные с сокетом
    - Записывает в поток вывода (пересылает на сервер) строку, а затем читает из потока ввода (принимает от сервера) ответ
    - Печатает ответ
---
## Клиент (TCP, java.net)
```java
import java.net.*;
import java.io.*;
public class TCPClient {
    public static void main(String args[]) {
        // arguments supply message and hostname
        int serverPort = 8080;
        try (Socket s = new Socket(args[1], serverPort)) {
            System.out.println("Connected to "+args[1]);
            DataInputStream in = new DataInputStream(s.getInputStream());
            DataOutputStream out = new DataOutputStream(s.getOutputStream());
            out.writeUTF(args[0]); // UTF is a string encoding
            String data = in.readUTF(); // read a line of data from the stream
            System.out.println("Received: " + data);
        } catch (UnknownHostException e) {
            System.out.println("Socket:" + e.getMessage()); // host cannot be resolved
        } catch (EOFException e) {
            System.out.println("EOF:" + e.getMessage()); // end of stream reached
        } catch (IOException e) {
            System.out.println("readline:" + e.getMessage()); // error in reading the stream
        }
    }
}
```
---
## Сервер (TCP, java.net) [начало]
```java
import java.net.*;
import java.io.*;
public class TCPServer {
    public static void main (String args[]) {
        try {
            int serverPort = 8080; // the server port
            ServerSocket listenSocket = new ServerSocket(serverPort); // new server port generated
            while(true) {
                Socket clientSocket = listenSocket.accept(); // listen for new connection
                ClientConnection c = new ClientConnection(clientSocket); // launch new thread
            }
        } catch(IOException e) { System.out.println("Listen socket:"+e.getMessage());
        }
    }
}
```
---
## Сервер (TCP, java.net) [окончание]
```java
class ClientConnection extends Thread {
 DataInputStream in;
 DataOutputStream out;
 Socket clientSocket;
 public Connection (Socket aClientSocket) {
  try {
   clientSocket = aClientSocket;
   in = new DataInputStream( clientSocket.getInputStream());
   out = new DataOutputStream( clientSocket.getOutputStream());
   this.start();
  } catch(IOException e){System.out.println("Connection:"+e.getMessage());
  }
 }
 public void run() { // an echo server
  try {   
   String data = in.readUTF(); // read a line of data from the stream 
   out.writeUTF(data); // write a line to the stream
   clientSocket.close();
  } catch (EOFException e){System.out.println("EOF:"+e.getMessage());
  } catch (IOException e) {System.out.println("readline:"+e.getMessage());
  }
 }
}
```
---
## Немного доработаем сервер
```java
package net.tcp;

import java.io.*;
import java.net.*;
import java.util.concurrent.*;

public class SmartTCPServer {
    public static final int N_THREADS = 10;
    public static final int PORT  = 8080;
    private static final Executor executor = Executors.newFixedThreadPool(N_THREADS);
    private static final BlockingQueue<Socket> connectionQueue= new LinkedBlockingQueue<>();

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(PORT);
        for (int i = 0; i < N_THREADS; i++ ){
            executor.execute(SmartTCPServer::run);
        }
        while (true) connectionQueue.add(serverSocket.accept());
    }

    private static void run() {
        while (true) {
            try {
                Socket socket = connectionQueue.take();
                System.out.println("Thread "+ Thread.currentThread() + " work with connection "+socket);
                var in = new DataInputStream(socket.getInputStream());
                var out = new DataOutputStream(socket.getOutputStream());
                String data = in.readUTF(); // read a line of data from the stream
                out.writeUTF(data); // write a line to the stream
                socket.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

---
## Полезные классы. URL (java.net)
- Класс java.net.URL представляет собой идентификатор ресурса – Uniform Resource Locator
- Включает в себя протокол и имя ресурса
- Имя ресурса:
    - Имя узла
    - Порт
    - Имя файла
    - Параметры
- Пример:
    - https://www.yandex.ru:443/search/?text=Java
---
## URL (java.net)
Создание абсолютного URL из строки
```java
URL url = new URL("https://www.yandex.ru/"); 
```
Создание абсолютного URL по частям
```java
URL yandex = new URL("https", “www.Yandex.ru", 443, “search/?text=java"); 
```
Создание относительного URL
```java
URL baseURL = new URL("https://www.Yandex.ru/search/"); 
URL search1URL = new URL(baseURL, "?text=Java"); 
URL search2URL = new URL(baseURL, " ?text=Oracle"); 
```
После создания URL не может быть изменен
---
## URL (java.net)
```java
public class UrlMain {
    public static void main(String[] args) throws Exception{
        try {
            URL url = new URL("https://www.yandex.ru:443/search/?text=Java");
            System.out.println(url.getQuery());
            System.out.println(url.getHost());
            System.out.println(url.getPort());
            System.out.println(url.getProtocol());
            try(BufferedReader br = new BufferedReader(
  new InputStreamReader(url.openStream()))){
                System.out.println(br.readLine());
            }catch(Exception e){
                e.printStackTrace();
            }
        } catch (MalformedURLException e) {
            e.printStackTrace();
        }
    }
}
```
---
## Передача данных в гетерогенных системах
Проблема передачи данных
- Информация, представленная как данные определяется внутри процесса
- Информация в сообщениях состоит только из последовательностей байтов
- Разные платформы по разному представляют примитивные типы
    - integers (big-endian &little-endian)
    - floating-point numbers
    - characters (ASCII & Unicode)
Данные должны быть упакованы перед передачей и восстановлены по прибытию
---
## Передача данных в гетерогенных системах
Представление данных
- Решение проблемы представления данных
    - Соглашение об использовании внешнего представления – два преобразования
    - Используется формат источника или приемщика – одно преобразование
- Передача структурированных типов
    - Типы данных могут не изменяться при передаче
    - Использование упакованных форматов (структуры «сплющиваются»)
- Форматы представления данных
    - SUN Microsystems XDR (eXternal Data Representation)
    - CORBA CDR (Common Data Representation)
    - ASN.1 (OSI layer 6)
- marshalling/unmarshalling
    - marshalling: преобразование исходных данных к виду, удобному для передачи
    - unmarshalling: восстановление исходных данных
    - Обычно выполняется промежуточным программным обеспечением (middleware)
---
## Предварительные итоги
- Пакет java.net предоставляет возможность работать с протоколами UDP и TCP
- Протокол TCP является надежным протоколом
    - Для  обмена данными используется механизм потоков (Stream)
- Протокол UDP является ненадежным протоколом
    - Обмен пакетами данных
    - Возможно использование прикладных протоколов поверх UDP, обеспечивающих нужный уровень надежности 
---
## Примеры
- Пример распределенной системы с использованием API java.net.
- Задачи:
    - Научиться передавать данные простых типов с использованием  API java.net
    - Научиться передавать данные сложных типов с использованием  API java.net и сериализации
---
## Описание задачи
Имеется сеть столовых. Необходимо разработать систему, автоматизирующую участок работы, связанный с обслуживанием пластиковых карт. В указанных столовых предусмотрен дополнительный сервис: постоянным клиентам выдают пластиковые карты, при предъявлении которых клиент получает существенную скидку. При этом на карту можно положить некоторую сумму денег и расплачиваться за обеды не наличными, а средствами, находящимися на карте. В целях экономии затрат на эмиссию карты закуплены не чиповые, а штриховые (такая карта несет только код – идентификатор клиента). Карта, выданная в одной столовой, может использоваться в другой столовой. Между столовыми нет устойчивых защищенных каналов связи, в связи с этим встает проблема передачи баланса карт между столовыми. Кроме того, в будущем, руководство предполагает изучать предпочтения клиентов, чтобы формировать более гибкую ценовую политику и более рационально управлять ассортиментом предлагаемых блюд.
---
## Обсуждение
- Операции
    - Выдача новой карты
    - Пополнение счета
    - Оплата покупки с помощью карты
    - Запрос баланса карты
- Ограничения
    - Одновременное обслуживание сервером нескольких столовых
    - Передача «пакетов» данных (связь неустойчивая)
---
## Обсуждение
- Архитектура системы
- Формат сообщения

---
## Класс BillingService (начало)
```java
package com.asw.net.ex1;
import java.net.*;
import java.util.HashMap;
import java.io.*;

public class BillingService extends Thread{
 public static final int ADD_NEW_CARD = 1;
 public static final int ADD_MONEY = 2;
 public static final int SUB_MONEY = 3;
 public static final int GET_CARD_BALANCE = 4;
 public static final int EXIT_CLIENT = 5;
 
 private int serverPort = 7896;
 private ServerSocket ss;
 private HashMap<String, Double> hash;
 
 public static void main(String[] args) {
  BillingService bs = new BillingService();
  bs.start();
 }
 public BillingService(){
  hash = new HashMap<>();
 }
```
---
## Класс BillingService (окончание)
```java
 public void run(){
     try {
  ss = new ServerSocket(serverPort);  System.out.println("Server started");
  while(true){
      Socket s = ss.accept();  System.out.println("Client accepted");
      BillingClientService bcs = new BillingClientService(this,
   new DataInputStream(s.getInputStream()),new            DataOutputStream(s.getOutputStream()));
      bcs.start();
  }
     } catch (IOException e) {e.printStackTrace();}
 }
 public void addNewCard(String personName, String card){ hash.put(card, new Double(0.0));}
 public void addMoney(String card, double money) { 
  Double d = hash.get(card);
  if (d!=null) hash.put(card,new Double(d.doubleValue()+money));}
 public void subMoney(String card, double money) {
  Double d = hash.get(card);
  if (d!=null) hash.put(card,new Double(d.doubleValue()-money));}
 public double getCardBalance(String card) {
  Double d = hash.get(card);
  if (d!=null) return d.doubleValue();  return 0;}
}
```
---
## Класс BillingClientService (начало)
```java
package com.asw.net.ex1;
import java.io.*;
public class BillingClientService extends Thread {
 DataInputStream dis;  DataOutputStream dos;  BillingService bs;
 public BillingClientService(BillingService bs,DataInputStream dis,DataOutputStream dos){
  this.bs = bs;  this.dis = dis;  this.dos = dos;
 }
 public void run(){  System.out.println("ClientService thread started");
  boolean work = true;
  while (work) { int command;
   try {  command = dis.readInt();
    switch (command) {
    case BillingService.ADD_NEW_CARD: addNewCard();  break;
    case BillingService.ADD_MONEY:         addMoney();  break;
    case BillingService.SUB_MONEY:         subMoney(); break;
    case BillingService.GET_CARD_BALANCE:  getCardBalance();break;
    case BillingService.EXIT_CLIENT:  work = false; break;
    default: System.out.println("Bad operation:" + command);
    }
   } catch (IOException e) {e.printStackTrace();}
  }
 }
```
---
## Класс BillingClientService (окончание)
```java
 void addNewCard() throws IOException{
  String personName = dis.readUTF();
  String card = dis.readUTF();
  bs.addNewCard(personName,card);
 }
 void addMoney() throws IOException{
  String card = dis.readUTF();
  double money = dis.readDouble();
  bs.addMoney(card,money);
 }
 void subMoney() throws IOException{
  String card = dis.readUTF();
  double money = dis.readDouble();
  bs.subMoney(card,money);
 }
 void getCardBalance() throws IOException{
  String card = dis.readUTF();
  double money = bs.getCardBalance(card);
  dos.writeDouble(money);
 }
}
```
---
## Класс BillingClient (начало)
```java
package com.asw.net.ex1;
import java.net.*;
import java.io.*;
public class BillingClient {
    int serverPort = 7896; String serverName; Socket s; DataInputStream dis; DataOutputStream dos;
    public BillingClient(String serverName){ this.serverName = serverName;}
    public static void main(String[] args) {
 BillingClient bc = new BillingClient(args[0]);
 try { bc.startTest(); } catch (IOException e) { e.printStackTrace(); }}
    public void startTest() throws IOException{
 connectToServer(); 
 sendNewCardOperation("Piter","1"); sendNewCardOperation("Stefan","2");     sendNewCardOperation("Nataly","3");
 for (int i = 0; i < 1000;i++){
     sendAddMoneyOperation("1", i%10);sendAddMoneyOperation("2", i%20);             sendAddMoneyOperation("3", i%30);
 }
 System.out.println("1:"+sendGetCardBalanceOperation("1")); System.out.println("2:"+sendGetCardBalanceOperation("2"));
 System.out.println("3:"+sendGetCardBalanceOperation("3")); closeConnection();}
    void connectToServer() throws UnknownHostException, IOException{
        s = new Socket(serverName, serverPort); 
        dis = new DataInputStream(s.getInputStream());
        dos = new DataOutputStream(s.getOutputStream());}  
```
---
## Класс BillingClient (окончание)
```java
 void sendNewCardOperation(String personName, String card) throws IOException{
  dos.writeInt(BillingService.ADD_NEW_CARD);
  dos.writeUTF(personName);
  dos.writeUTF(card);
 }
 void sendAddMoneyOperation(String card, double money) throws IOException{
  dos.writeInt(BillingService.ADD_MONEY);
  dos.writeUTF(card);
  dos.writeDouble(money);
 }
 void sendSubMoneyOperation(String card, double money) throws IOException{
  dos.writeInt(BillingService.SUB_MONEY);
  dos.writeUTF(card);
  dos.writeDouble(money);
 }
 double sendGetCardBalanceOperation(String card) throws IOException{
  dos.writeInt(BillingService.GET_CARD_BALANCE);
  dos.writeUTF(card);
  return dis.readDouble();
 }
 void closeConnection() throws IOException{
  dos.writeInt(BillingService.EXIT_CLIENT);
 }
}
```
---
## Обсуждение результатов
- Используемый прикладной протокол
    - Низкая надежность
    - Сложность поддержки
- Используемый механизм параллельной обработки 
    - Непредсказуемые (в общем виде - НЕПРАВИЛЬНЫЕ) результаты из-за «гонок потоков»
- Вывод: приложение работает неудовлетворительно
- Решение:
    - Другой прикладной протокол
    - Механизм блокировки ресурсов
---
## Прикладной протокол
- Сериализация Java
    - Сериализуются значения полей
    - Необходимо реализовывать интерфейс Serializable
    - Интерфейс Serializable - тэгирующий
---
## Классы – «сообщения»
- Карта (владелец; дата выдачи; номер карты; баланс )
- Операция по изменению баланса (номер карты; сумма; дата операции)
---
## Класс Card
```java
package com.asw.net.ex2;
import java.io.Serializable;
import java.util.*;

public class Card implements Serializable{
 public Card(String person, Date createDate, String cardNumber,double balance){
  this.person = person;
  this.createDate = createDate;
  this.cardNumber = cardNumber;
  this.balance = balance;
 }
 public final String person;
 public final transient Date createDate;
 public final String cardNumber;
 public final double balance;
 public String toString(){
  return "Card:cardNumber="+cardNumber+"\tBalance=“
   +balance+"\tPerson="+person+"\tCreateDate="+createDate+"";
 }
}
```
---
## Класс CardOperation
```java
package com.asw.net.ex2;
import java.util.*;
import java.io.*;


public class CardOperation implements Serializable {
 public CardOperation(String card,double amount,Date operationDate){
  this.card = card;
  this.amount = amount;
  this.operationDate = operationDate;
 }
 public final String card;
 public final double amount;
 public final Date operationDate;
}
```
---
## Класс BillingService (начало)
```java
package com.asw.net.ex2;
import java.net.*;
import java.util.HashMap;
import java.io.*;

public class BillingService extends Thread{
 private int serverPort = 7896;  private ServerSocket ss;  private HashMap<String, Card> hash;
 public static void main(String[] args) {
  BillingService bs = new BillingService(); bs.start();
 }
 public BillingService(){  hash = new HashMap<>(); }
 public void run(){
  try { ss = new ServerSocket(serverPort);
   System.out.println("Server started");
   while(true){ System.out.println("new client waiting...");
    Socket s = ss.accept();
    System.out.println("Client accepted");
    BillingClientService bcs = new BillingClientService(this,s);
    System.out.println("bcs created");
    bcs.start();
   }
  } catch (IOException e) { e.printStackTrace(); }
  
 }
```
---
## Класс BillingService (окончание)
```java
 public void addNewCard(Card card) {
  hash.put(card.cardNumber, card);
 }

 public void addMoney(String card, double money) {
  Card c = hash.get(card);
  if (c==null) {
   System.out.println("Bad Card number\n");
   return;
  };
  c.balance+=money;
  hash.put(card,c);
 }

 public Card getCard(String card){
  return hash.get(card);
 }
 
}
```
---
## Класс BillingClientService
```java
package com.asw.net.ex2;
import java.io.*;
import java.net.*;
public class BillingClientService extends Thread {
    ObjectInputStream ois;  ObjectOutputStream oos;   BillingService bs;   Socket s;
    public BillingClientService(BillingService bs,Socket s){
 System.out.println("Constructor BillingClientService\n");
 this.bs = bs;  this.s = s;
 try {  this.oos = new ObjectOutputStream(s.getOutputStream());  
                 this.ois = new ObjectInputStream(s.getInputStream());
 } catch (IOException e) {  e.printStackTrace(); }
    }
    public void run(){
 System.out.println("ClientService thread started\n");  boolean work = true;
 while (work) { int command;      Object o;
             try { o = ois.readObject();
  if (o instanceof Card[]) {Card[] cards = (Card[])o;
   for (int i=0;i<cards.length;i++){  bs.addNewCard(cards[i]); }
  }else if (o instanceof CardOperation[]){  CardOperation[] co = (CardOperation[])o;
   for (int i=0;i<co.length;i++){  bs.addMoney(co[i].card,co[i].amount); }
  }else if (o instanceof String){  oos.writeObject(bs.getCard((String)o)); 
  }else System.out.println("Bad operation");
             } catch (IOException e) {  e.printStackTrace();
     } catch (ClassNotFoundException e) { e.printStackTrace(); }
 }
    }
}
```
---
## Класс BillingClient (начало)
```java
package com.asw.net.ex2;
import java.net.*;
import java.util.Date;
import java.io.*;
public class BillingClient {
    int serverPort = 7896; String serverName; Socket s; ObjectInputStream ois;ObjectOutputStream oos;
    public BillingClient(String serverName){  this.serverName = serverName;  }
    public void startTest() throws IOException{
 connectToServer();
 Card[] cards = {new Card("Piter",new Date(),"1",0.0), new Card("Stefan",new Date(),"2",0.0)
  ,new Card("Nataly",new Date(),"3",0.0)};
 processCard(cards);
 int cnt = 30000;  CardOperation[] co = new CardOperation[cnt];
 for (int i = 0; i < cnt; i++) { switch (i%3){
  case 0: co[i] = new CardOperation("1",1,new Date());break;
  case 1: co[i] = new CardOperation("2",2,new Date());break;
  case 2: co[i] = new CardOperation("3",3,new Date());break;
          }
 }
 processOperation(co);
 try { System.out.println("getCard: "+getCard("1")); System.out.println("getCard: "+getCard("2"));   System.out.println("getCard: "+getCard("3"));
 } catch (IOException e1) {e1.printStackTrace();
 } catch (ClassNotFoundException e1) {e1.printStackTrace();}
      
    }
    void connectToServer() throws UnknownHostException, IOException{
 s = new Socket(serverName, serverPort); System.out.println("connection established\n");
 ois = new ObjectInputStream(s.getInputStream());  oos = new ObjectOutputStream(s.getOutputStream());
 } 
```
---
## Класс BillingClient (окончание)
```java
 void processOperation(CardOperation[] co) throws IOException{
  System.out.println(co);
  oos.writeObject(co);
 }
 void processCard(Card[] c) throws IOException{
  System.out.println("processCard: c="+c);
  oos.writeObject(c);
 }
 Card getCard(String card) throws IOException, ClassNotFoundException{
  oos.writeObject(card);
  return (Card)ois.readObject();
 }
 public static void main(String[] args) throws Exception{
  BillingClient bc = new BillingClient(args[0]);
  try {
   bc.startTest();
  } catch (IOException e) {
   e.printStackTrace();
  }
   
 }
}
```
---
## Итоги
- API java.net позволяет реализовывать распределенные приложения
- Выбор прикладного протокола – важная часть проектирования 
- Обеспечение корректной работы в параллельной среде – необходимая часть реализации распределенного приложения
- Сериализация – встроенный механизм Java, обеспечивающий широкие возможности по передаче данных сложной структуры

