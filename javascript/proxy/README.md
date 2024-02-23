### Proxy 

- Объект Proxy позволяет создать прокси для другого объекта, может перехватывать и переопределить основные операции для данного объекта.

- Прокси используются программистами для объявления расширенной семантики JavaScript объектов. Стандартная семантика реализована в движке JavaScript, который обычно написан на низкоуровневом языке программирования, например C++. Прокси позволяют программисту определить поведение объекта при помощи JavaScript. Другими словами они являются инструментом метапрограммирования.

- Примечание: реализация прокси в SpiderMonkey является прототипом, в котором прокси API и семантика не стабильны. Также, реализация в SpiderMonkey может не соответствовать последней версии спецификации. Она может быть изменена в любой момент и предоставляется исключительно как экспериментальная функция. Не полагайтесь на неё в производственном коде.

- Эта страница описывает новый API (называемый «непосредственным проксированием»), который является частью Firefox 18. Для просмотра старого API (Firefox 17 и ниже) посетите страницу описания старого прокси API.



- прокси (proxy)
- Объект, оборачивающий исходный объект.

- обработчик (handler)
- Объект-заменитель, содержащий ловушки. Определяет, какие операции будут перехвачены, также переопределяет перехваченные операции.

- ловушки (traps)
- Методы, которые предоставляют доступ к свойствам. Это аналогично концепции ловушек в операционных системах.

- цель (target)
- Исходный объект, который виртуализируется прокси. Он часто используется в качестве источника данных в прокси. Для него проверяются инварианты относительно расширяемости и настраиваемости свойств.

- Прокси - это новые объекты; невозможно выполнить "проксирование" существующего объекта. Пример создания прокси:
```javascript
var p = new Proxy(target, handler);
```
Где:

- target — исходный объект (может быть объектом любого типа, включая массив, функцию и даже другой прокси объект).
- handler — объект-обработчик, методы (ловушки) которого определяют поведение прокси во время выполнения операции над ним.

- Все ловушки опциональны. В случае, если ловушка не задана, то стандартным поведением будет перенаправление операции к объекту-цели.

- Несмотря на то, что прокси предоставляют много возможностей пользователям, некоторые операции не перехватываются для сохранения постоянства языка:

- Простой и строгий оператор равенства (==, ===) не перехватывается. p1 === p2 равны, только если p1 и p2 ссылаются на один и тот же прокси.
Текущая реализация Object.getPrototypeOf(proxy) всегда возвращает Object.getPrototypeOf(target), потому что в ES2015 перехватчик getPrototypeOf пока не реализован.
typeof proxy всегда возвращает typeof target. В частности, proxy может быть использован как функция только если target является функцией.
Array.isArray(proxy) всегда возвращает Array.isArray(target).
Object.prototype.toString.call(proxy) всегда возвращает Object.prototype.toString.call(target), потому что в ES2015 перехватчик Symbol.toStringTag пока не реализован.

**Простой пример**
Объект, возвращающий значение 37, в случае отсутствия свойства с указанным именем:
```javascript
var handler = {
  get: function (target, name) {
    return name in target ? target[name] : 37;
  },
};

var p = new Proxy({}, handler);
p.a = 1;
p.b = undefined;

console.log(p.a, p.b); // 1, undefined
console.log("c" in p, p.c); // false, 37
``` 

- Перенаправляющий прокси
- В данном примере мы используем JavaScript объект, к которому наш прокси направляет все запросы: 

```javascript
var target = {};
var p = new Proxy(target, {});

p.a = 37; // операция перенаправлена прокси

console.log(target.a); // 37. Операция была успешно перенаправлена
```

- **Проверка**
- При помощи Proxy вы можете легко проверять передаваемые объекту значения: 

```javascript
let validator = {
  set: function (obj, prop, value) {
    if (prop === "age") {
      if (!Number.isInteger(value)) {
        throw new TypeError("The age is not an integer");
      }
      if (value > 200) {
        throw new RangeError("The age seems invalid");
      }
    }

    // Стандартное сохранение значения
    obj[prop] = value;

    // Обозначить успех
    return true;
  },
};

let person = new Proxy({}, validator);

person.age = 100;
console.log(person.age); // 100
person.age = "young"; // Вызовет исключение
person.age = 300; // Вызовет исключение
``` 

- **Дополнение конструктора**
- Функция прокси может легко дополнить конструктор новым:

```javascript
function extend(sup, base) {
  var descriptor = Object.getOwnPropertyDescriptor(
    base.prototype,
    "constructor",
  );

  const prototype = { ...base.prototype };

  base.prototype = Object.create(sup.prototype);
  base.prototype = Object.assign(base.prototype, prototype);

  var handler = {
    construct: function (target, args) {
      var obj = Object.create(base.prototype);
      this.apply(target, obj, args);
      return obj;
    },
    apply: function (target, that, args) {
      sup.apply(that, args);
      base.apply(that, args);
    },
  };
  var proxy = new Proxy(base, handler);
  descriptor.value = proxy;
  Object.defineProperty(base.prototype, "constructor", descriptor);
  return proxy;
}

var Person = function (name) {
  this.name = name;
};

var Boy = extend(Person, function (name, age) {
  this.age = age;
});

Boy.prototype.sex = "M";

var Peter = new Boy("Peter", 13);
console.log(Peter.sex); // "M"
console.log(Peter.name); // "Peter"
console.log(Peter.age); // 13
``` 

- **Манипуляция DOM элементами**
- Иногда возникает необходимость переключить атрибут или имя класса у двух разных элементов: 

```javascript
let view = new Proxy(
  {
    selected: null,
  },
  {
    set: function (obj, prop, newval) {
      let oldval = obj[prop];

      if (prop === "selected") {
        if (oldval) {
          oldval.setAttribute("aria-selected", "false");
        }
        if (newval) {
          newval.setAttribute("aria-selected", "true");
        }
      }

      // Стандартное сохранение значения
      obj[prop] = newval;
    },
  },
);

let i1 = (view.selected = document.getElementById("item-1"));
console.log(i1.getAttribute("aria-selected")); // 'true'

let i2 = (view.selected = document.getElementById("item-2"));
console.log(i1.getAttribute("aria-selected")); // 'false'
console.log(i2.getAttribute("aria-selected")); // 'true'
``` 

- **Изменение значений и дополнительные свойства**
- Прокси объект products проверяет передаваемые значения и преобразует их в массив в случае необходимости. Объект также поддерживает дополнительное свойство latestBrowser на чтение и запись.

```javascript
let products = new Proxy(
  {
    browsers: ["Internet Explorer", "Netscape"],
  },
  {
    get: function (obj, prop) {
      // Дополнительное свойство
      if (prop === "latestBrowser") {
        return obj.browsers[obj.browsers.length - 1];
      }

      // Стандартный возврат значения
      return obj[prop];
    },
    set: function (obj, prop, value) {
      // Дополнительное свойство
      if (prop === "latestBrowser") {
        obj.browsers.push(value);
        return;
      }

      // Преобразование значения, если оно не массив
      if (typeof value === "string") {
        value = [value];
      }

      // Стандартное сохранение значения
      obj[prop] = value;
    },
  },
);

console.log(products.browsers); // ['Internet Explorer', 'Netscape']
products.browsers = "Firefox"; // передаётся как строка (по ошибке)
console.log(products.browsers); // ['Firefox'] <- проблем нет, значение - массив

products.latestBrowser = "Chrome";
console.log(products.browsers); // ['Firefox', 'Chrome']
console.log(products.latestBrowser); // 'Chrome'
``` 
- **Поиск элемента массива по его свойству**
- Данный прокси расширяет массив дополнительными возможностями. Как вы видите, вы можете гибко "задавать" свойства без использования Object.defineProperties (en-US). Данный пример также может быть использован для поиска строки таблицы по её ячейке. В этом случае целью будет table.rows (en-US).

```javascript
let products = new Proxy(
  [
    { name: "Firefox", type: "browser" },
    { name: "SeaMonkey", type: "browser" },
    { name: "Thunderbird", type: "mailer" },
  ],
  {
    get: function (obj, prop) {
      // Стандартное возвращение значения; prop обычно является числом
      if (prop in obj) {
        return obj[prop];
      }

      // Получение количества продуктов; псевдоним products.length
      if (prop === "number") {
        return obj.length;
      }

      let result,
        types = {};

      for (let product of obj) {
        if (product.name === prop) {
          result = product;
        }
        if (types[product.type]) {
          types[product.type].push(product);
        } else {
          types[product.type] = [product];
        }
      }

      // Получение продукта по имени
      if (result) {
        return result;
      }

      // Получение продуктов по типу
      if (prop in types) {
        return types[prop];
      }

      // Получение типов продуктов
      if (prop === "types") {
        return Object.keys(types);
      }

      return undefined;
    },
  },
);

console.log(products[0]); // { name: 'Firefox', type: 'browser' }
console.log(products["Firefox"]); // { name: 'Firefox', type: 'browser' }
console.log(products["Chrome"]); // undefined
console.log(products.browser); // [{ name: 'Firefox', type: 'browser' }, { name: 'SeaMonkey', type: 'browser' }]
console.log(products.types); // ['browser', 'mailer']
console.log(products.number); // 3
``` 

- **Пример использования всех перехватчиков**
- В данном примере, использующем все виды перехватчиков, мы попытаемся проксировать не нативный объект, который частично приспособлен для этого - docCookies, созданном в разделе "little framework" и опубликованном на странице document.cookie (en-US).

```javascript
/*
  var docCookies = ... получить объект "docCookies" можно здесь:
  https://developer.mozilla.org/ru/docs/DOM/document.cookie#A_little_framework.3A_a_complete_cookies_reader.2Fwriter_with_full_unicode_support
*/

var docCookies = new Proxy(docCookies, {
  get: function (oTarget, sKey) {
    return oTarget[sKey] || oTarget.getItem(sKey) || undefined;
  },
  set: function (oTarget, sKey, vValue) {
    if (sKey in oTarget) {
      return false;
    }
    return oTarget.setItem(sKey, vValue);
  },
  deleteProperty: function (oTarget, sKey) {
    if (sKey in oTarget) {
      return false;
    }
    return oTarget.removeItem(sKey);
  },
  enumerate: function (oTarget, sKey) {
    return oTarget.keys();
  },
  iterate: function (oTarget, sKey) {
    return oTarget.keys();
  },
  ownKeys: function (oTarget, sKey) {
    return oTarget.keys();
  },
  has: function (oTarget, sKey) {
    return sKey in oTarget || oTarget.hasItem(sKey);
  },
  hasOwn: function (oTarget, sKey) {
    return oTarget.hasItem(sKey);
  },
  defineProperty: function (oTarget, sKey, oDesc) {
    if (oDesc && "value" in oDesc) {
      oTarget.setItem(sKey, oDesc.value);
    }
    return oTarget;
  },
  getPropertyNames: function (oTarget) {
    return Object.getPropertyNames(oTarget).concat(oTarget.keys());
  },
  getOwnPropertyNames: function (oTarget) {
    return Object.getOwnPropertyNames(oTarget).concat(oTarget.keys());
  },
  getPropertyDescriptor: function (oTarget, sKey) {
    var vValue = oTarget[sKey] || oTarget.getItem(sKey);
    return vValue
      ? {
          value: vValue,
          writable: true,
          enumerable: true,
          configurable: false,
        }
      : undefined;
  },
  getOwnPropertyDescriptor: function (oTarget, sKey) {
    var vValue = oTarget.getItem(sKey);
    return vValue
      ? {
          value: vValue,
          writable: true,
          enumerable: true,
          configurable: false,
        }
      : undefined;
  },
  fix: function (oTarget) {
    return "not implemented yet!";
  },
});

/* Проверка cookies */

alert((docCookies.my_cookie1 = "First value"));
alert(docCookies.getItem("my_cookie1"));

docCookies.setItem("my_cookie1", "Changed value");
alert(docCookies.my_cookie1);
```

- Proxy and Reflect
- A Proxy object wraps another object and intercepts operations, like reading/writing properties and others, optionally handling them on its own, or transparently allowing the object to handle them.

- Proxies are used in many libraries and some browser frameworks. We’ll see many practical applications in this article. 

- **Proxy**
- The syntax:
```javascript
let proxy = new Proxy(target, handler);
``` 

- target – is an object to wrap, can be anything, including functions.
handler – proxy configuration: an object with “traps”, methods that intercept operations. – e.g. get trap for reading a property of target, set trap for writing a property into target, and so on.
For operations on proxy, if there’s a corresponding trap in handler, then it runs, and the proxy has a chance to handle it, otherwise the operation is performed on target.

As a starting example, let’s create a proxy without any traps: