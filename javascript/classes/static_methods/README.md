### Статические свойства и методы 

- Мы также можем присвоить метод самому классу. Такие методы называются статическими.

- В объявление класса они добавляются с помощью ключевого слова static, например:

```javascript
class User {
  static staticMethod() {
    alert(this === User);
  }
}

User.staticMethod(); // true
``` 

- Это фактически то же самое, что присвоить метод напрямую как свойство функции:
```javascript
class User { }

User.staticMethod = function() {
  alert(this === User);
};
``` 

- Значением this при вызове User.staticMethod() является сам конструктор класса User (правило «объект до точки»).

- Обычно статические методы используются для реализации функций, которые будут принадлежать классу в целом, но не какому-либо его конкретному объекту.

- Звучит не очень понятно? Сейчас все встанет на свои места.

- Например, есть объекты статей Article, и нужна функция для их сравнения.

- Естественное решение – сделать для этого статический метод Article.compare:

```javascript
class Article {
  constructor(title, date) {
    this.title = title;
    this.date = date;
  }

  static compare(articleA, articleB) {
    return articleA.date - articleB.date;
  }
}

// использование
let articles = [
  new Article("HTML", new Date(2019, 1, 1)),
  new Article("CSS", new Date(2019, 0, 1)),
  new Article("JavaScript", new Date(2019, 11, 1))
];

articles.sort(Article.compare);

alert( articles[0].title ); // CSS
``` 

- Здесь метод Article.compare стоит «над» статьями, как средство для их сравнения. Это метод не отдельной статьи, а всего класса.

- Другим примером может быть так называемый «фабричный» метод.

- Скажем, нам нужно несколько способов создания статьи:

1. Создание через заданные параметры (title, date и т. д.).
2. Создание пустой статьи с сегодняшней датой.
3. …или как-то ещё.
- Первый способ может быть реализован через конструктор. А для второго можно использовать статический метод класса.

- Такой как Article.createTodays() в следующем примере:
```javascript
class Article {
  constructor(title, date) {
    this.title = title;
    this.date = date;
  }

  static createTodays() {
    // помним, что this = Article
    return new this("Сегодняшний дайджест", new Date());
  }
}

let article = Article.createTodays();

alert( article.title ); // Сегодняшний дайджест
```

- Теперь каждый раз, когда нам нужно создать сегодняшний дайджест, нужно вызывать Article.createTodays(). Ещё раз, это не метод одной статьи, а метод всего класса.

- Статические методы также используются в классах, относящихся к базам данных, для поиска/сохранения/удаления вхождений в базу данных, например:
```javascript
// предположим, что Article - это специальный класс для управления статьями
// статический метод для удаления статьи по id:
Article.remove({id: 12345});
```
- Статические методы недоступны для отдельных объектов
- Статические методы могут вызываться для классов, но не для отдельных объектов.

- Например. такой код не будет работать:
```javascript
// ...
article.createTodays(); /// Error: article.createTodays is not a function
``` 

### Статические свойства

- Эта возможность была добавлена в язык недавно. Примеры работают в последнем Chrome.
- Статические свойства также возможны, они выглядят как свойства класса, но с static в начале:
```javascript
class Article {
  static publisher = "Илья Кантор";
}

alert( Article.publisher ); // Илья Кантор
``` 

Это то же самое, что и прямое присваивание Article:
```javascript
Article.publisher = "Илья Кантор";
``` 
- Наследование статических свойств и методов
- **Статические свойства и методы наследуются.**

- Например, метод Animal.compare в коде ниже наследуется и доступен как Rabbit.compare:
```javascript
class Animal {

  constructor(name, speed) {
    this.speed = speed;
    this.name = name;
  }

  run(speed = 0) {
    this.speed += speed;
    alert(`${this.name} бежит со скоростью ${this.speed}.`);
  }

  static compare(animalA, animalB) {
    return animalA.speed - animalB.speed;
  }

}

// Наследует от Animal
class Rabbit extends Animal {
  hide() {
    alert(`${this.name} прячется!`);
  }
}

let rabbits = [
  new Rabbit("Белый кролик", 10),
  new Rabbit("Чёрный кролик", 5)
];

rabbits.sort(Rabbit.compare);

rabbits[0].run(); // Чёрный кролик бежит со скоростью 5.
``` 

- Мы можем вызвать Rabbit.compare, при этом будет вызван унаследованный Animal.compare.

- Как это работает? Снова с использованием прототипов. Как вы уже могли предположить, extends даёт Rabbit ссылку [[Prototype]] на Animal. 

- Так что Rabbit extends Animal создаёт две ссылки на прототип:

- Функция Rabbit прототипно наследует от функции Animal.
Rabbit.prototype прототипно наследует от Animal.prototype.
В результате наследование работает как для обычных, так и для статических методов.

- Давайте это проверим кодом:
```javascript
class Animal {}
class Rabbit extends Animal {}

// для статики
alert(Rabbit.__proto__ === Animal); // true

// для обычных методов
alert(Rabbit.prototype.__proto__ === Animal.prototype); // true
``` 

### Итого
- Статические методы используются для функциональности, принадлежат классу «в целом», а не относятся к конкретному объекту класса.

- Например, метод для сравнения двух статей Article.compare(article1, article2) или фабричный метод Article.createTodays().

- В объявлении класса они помечаются ключевым словом static.

- Статические свойства используются в тех случаях, когда мы хотели бы сохранить данные на уровне класса, а не какого-то одного объекта.

- Синтаксис:
```javascript
class MyClass {
  static property = ...;

  static method() {
    ...
  }
}
``` 

- Технически, статическое объявление – это то же самое, что и присвоение классу:
```javascript
MyClass.property = ...
MyClass.method = ...
``` 

- Статические свойства и методы наследуются.

- Для class B extends A прототип класса B указывает на A: B.[[Prototype]] = A. Таким образом, если поле не найдено в B, поиск продолжается в A.

### Класс расширяет объект?
- Как мы уже знаем, все объекты наследуют от Object.prototype и имеют доступ к «общим» методам объекта, например hasOwnProperty.

```javascript
class Rabbit {
  constructor(name) {
    this.name = name;
  }
}

let rabbit = new Rabbit("Rab");

// метод hasOwnProperty от Object.prototype
alert( rabbit.hasOwnProperty('name') ); // true
``` 

- Но что если мы явно напишем "class Rabbit extends Object" – тогда результат будет отличаться от обычного "class Rabbit"?

- В чем разница?

- Ниже пример кода с таким наследованием (почему он не работает? исправьте его): 

```javascript
class Rabbit extends Object {
  constructor(name) {
    this.name = name;
  }
}

let rabbit = new Rabbit("Кроль");

alert( rabbit.hasOwnProperty('name') ); // Ошибка
``` 

- Сперва давайте разберёмся, почему код не работает.

- Причина становится очевидна, если мы попытаемся запустить его. Унаследованный конструктор класса должен вызывать super(). В противном случае "this" будет не определён.

- Решение: 

```javascript
class Rabbit extends Object {
  constructor(name) {
    super(); // надо вызвать конструктор родителя, когда наследуемся
    this.name = name;
  }
}

let rabbit = new Rabbit("Кроль");

alert( rabbit.hasOwnProperty('name') ); // true
``` 

- Но это ещё не все.

- Даже после исправления есть важное различие между "class Rabbit extends Object" и class Rabbit.

- Как мы знаем, синтаксис «extends» устанавливает 2 прототипа:

- Между "prototype" функций-конструкторов (для методов)
Между самими функциями-конструкторами (для статических методов).
В случае с class Rabbit extends Object это значит: 

```javascript
class Rabbit extends Object {}

alert( Rabbit.prototype.__proto__ === Object.prototype ); // (1) true
alert( Rabbit.__proto__ === Object ); // (2) true
``` 

- Таким образом, Rabbit предоставляет доступ к статическим методам Object через Rabbit, например:

```javascript
class Rabbit extends Object {}

// обычно мы вызываем Object.getOwnPropertyNames
alert( Rabbit.getOwnPropertyNames({a: 1, b: 2}) ); // a,b
``` 

- Но если явно не наследуем от объекта, то для Rabbit.__proto__ не установлено значение Object.

- Пример: 
```javascript
class Rabbit {}

alert( Rabbit.prototype.__proto__ === Object.prototype ); // (1) true
alert( Rabbit.__proto__ === Object ); // (2) false (!)
alert( Rabbit.__proto__ === Function.prototype ); // как у каждой функции по умолчанию

// ошибка - нет такой функции у Rabbit
alert( Rabbit.getOwnPropertyNames({a: 1, b: 2}) ); // Ошибка
``` 

- Таким образом, в этом случае у Rabbit нет доступа к статическим методам Object.

- Кстати, у Function.prototype также есть «общие» методы, такие как call, bind и т. д. Они в конечном итоге доступны в обоих случаях, потому что для встроенного конструктора Object Object.__proto__ === Function.prototype. 

