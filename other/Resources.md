#### Введение
Часто бывает нужно использовать какие-то ресурсы в работе вашего приложения.
Например, вы хотите использовать какую-то картинку, внешний файл с данными или вообще файл с настройками.

Для такого подхода нам надо как-то указать наш ресурс.
Как же быть?

Есть несколько возможных вариантов:
* Использование абсолютного пути на диске.
* Использование `java.lang.Class` методов, таких как `getResourceAsStream`, `getResource` и т.д

#### Абсолютный путь до ресурса
Решение не самое лучшее и годится лишь для тестовых вещей, сделанных на коленке. За исключением, быть может,
каких то внещних ресурсов, которые не меняют своего расположения.

Но даже в таком случае лучше указывать расположение таких ресурсов в каком-нибудь файле с настройками, а не 'зашивать' явно.

#### Использование методов `java.lang.Class`
Как видно, методы `getResource(String)` и `getResourceAsStream(String)` ждут от нас какой-то строки.
Эта строка - имя ресурса - это путь к ресурсу и его имя.

Кажется все просто, но тут есть небольшой подводный камень, а именно, что имя ресурса можно по разному интерпретировать.

Итак:
* Абсолютное имя, начинается с символа '/'.
* Относительное имя пишется как есть.

В чем же разница?
А в том, что в первом случае ресурс мы ищем относительно корня `classpath`, а во втором - к имени ресурса приписывается еще и *имя пакета*, в котором находится *текущий* класс.

Т.е пусть у нас есть класс `Example`, находящийся в пакете `com.github.aarexer` и мы имеем ресурс `test.txt`, тогда
```java
Example.class.getResource("/test.txt")
```

Мы будем искать по `classpath/test.txt`, а если:
```java
Example.class.getResource("test.txt")
```
Будет `classpath/com/github/aarexer/test.txt`

Где `classpath` - это ваш заданный `classpath`.

Два примера использования:
//todo example

//todo issue

#### Работа с jar
Помните, что если вы читаете ресурс из jar и используете '/', то необходимо использовать:
```java
getResourceAsStream(...);
```

Иначе вы получите `java.lang.IllegalArgumentException`