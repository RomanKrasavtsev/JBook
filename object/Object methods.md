##Введение
Класс Object является корнем в иерархии классов в Java. Все объекты, **включая массивы**, наследуют методы Object.

Что за методы?

Методы Object-а:
* `.toString()`
* `.clone()`
* `.hashCode()`
* `.wait()`
* `.notify()`
* `.notifyAll()`
* `.equals()`
* `.finalize()`
* `.getClass()`

Подробнее о каждом.
### toString
Зачем метод нужен - понятно. Строковое представление объекта, также участвует в конкатенации.
```java
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```
По умолчанию он включает имя класса и hashCode в hexademical представлении , разделенные "@".
Мы можем переопределить этот метод.

В JavaDoc рекомендуется этот метод переопределять.

Подробнее про: [toString](./ToString.md)

### finalize()
Когда вызывается finalize?

обратимся к JavaDoc
>Called by the garbage collector on an object when garbage collection determines that there are no more references to the object.

Может показаться, что это отличный метод, в который хорошо поместить закрытие ресурсов и прочее, что нужно освобождать/удалять при уничтожении класса. Но это **неправильно**.

И прежде всего потому что у нас нет **НИКАКИХ** гарантий, что этот метод вообще вызовется. Например, у нас где-то в коде есть забытая ссылка на объект, и все - мы про ссылку ничего не знаем, она существует и GC не будет удалять этот объект. А в некоторых случаях этот метод вообще не вызовется! Например, если наше приложение вдруг упадет или мы его остановим.

Еще одно **заблуждение**  - если мы в finalize() методе повесим процесс(ведь GC работает в фоне), то и наш GC перестанет работать.
Это тоже **не верно**.

А вот почему:
Наш GC не вызывает `finalize()` напрямую, а добавляет объекты в список, вызывая `java.lang.ref.Finalizer.register(Object)`. Объект класса Finalizer представляет собой ссылку на объект, для которого надо вызвать finalize(), и хранит ссылки на следующий и предыдущий Finalizer, формируя двусвязный список.
Вызов же finalize() происходит в отдельном потоке `java.lang.ref.Finalizer.FinalizerThread`, вызовы finalize идут последовательно так, как добавлялись. Поэтому, если finalize зависает - виснет именно FinalizerThread, если объекты не имеют finalize(он пустой), то они будут и дальше удаляться, если же нет - то добавляются в список и ждут, пока все не отвиснет или не упадет окончательно, либо мы вообще не закроем наше приложение.

Как выглядит finalize() в openJDK:
```java
protected void finalize() throws Throwable { }
```
Не рекомендуется его переопределять и пихать туда какую-то логику, насколько я смог найти информацию об этом.
Но в очень редких случаях - этот метод может помочь.
Например, у нас объект с Weak/Soft References и мы работаем с реально большим файлом. В какой-то момент нам не хватает памяти и GC начинает вычищать объекты по нашим ссылкам и убирает наш объект, но мы **ОБЯЗАНЫ** закрыть наш файловый дескриптор. И вот в таком случае - переопределение метода поможет нам.
Но вместо такого можно использовать `PhantomReference `.

В заключении: Лучше наверное не переопределять этот метод, так как больше проблем, чем пользы от него может быть в обычных случаях.

### getClass()
Возвращает класс объекта.

Выглядит как:
```java
public final native Class<?> getClass();
```

Используется для Reflection API.

### hashCode()
Ясно из названия, что `hashCode()` вернет нам hash code объекта.
В Oracle JDK - это native method.

```java
 public native int hashCode();
```

Метод крайне важный и часто используемый. Используется он и в сравнении объектов, и в работе с hash- таблицами(мапами).

Важно понимать:
Если два Objects имеют разные hash code - это значит, что у нас разные объекты!
Однако если у двух Objects одинаковые hash code - это вовсе не означает еще, что это один и тот же объект.
Такая ситуация - когда разные объекты имеют одинаковые hash code - называется `collision` или коллизией.

Но!
Напишем вот такое:
```java
class A {
   private int a;
   public A(int a) { this.a = a; }
}

//code

print(new A(4).hashCode());
print(new A(4).hashCode());
```

И получим, что два объекта с одинаковыми полями имеют разный hash code. Почему? Потому что мы не переопределили  `hashCode()` метод, вызвав  native method из `Object`, который сигнализирует нам, о том, что у нас два разных объекта, что вполне логично.

Важно знать также, что Integer и типы обертки возвращают в качестве hash code свое значение.

Подробнее про hashCode: [HashCode](./Hashcode.md)

### equals
Подробнее про [equals](./Equals.md)