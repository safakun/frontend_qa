### debounce / throttle 

- Тормозящий (throttling) декоратор

- Создайте «тормозящий» декоратор throttle(f, ms), который возвращает обёртку.

- При многократном вызове он передает вызов f не чаще одного раза в ms миллисекунд.

- По сравнению с декоратором debounce поведение совершенно другое:

- **debounce запускает функцию один раз после периода «бездействия». Подходит для обработки конечного результата.**
- **throttle запускает функцию не чаще, чем указанное время ms. Подходит для регулярных обновлений, которые не должны быть слишком частыми.**
- Другими словами, throttle похож на секретаря, который принимает телефонные звонки, но при этом беспокоит начальника (вызывает непосредственно f) не чаще, чем один раз в ms миллисекунд.

- Давайте рассмотрим реальное применение, чтобы лучше понять это требование и выяснить, откуда оно взято.

- Например, мы хотим отслеживать движения мыши.

- В браузере мы можем реализовать функцию, которая будет запускаться при каждом перемещении указателя и получать его местоположение. Во время активного использования мыши эта функция запускается очень часто, что-то около 100 раз в секунду (каждые 10 мс). Мы бы хотели обновлять некоторую информацию на странице при передвижении указателя.

- …Но функция обновления update() слишком ресурсоёмкая, чтобы делать это при каждом микродвижении. Да и нет смысла делать обновление чаще, чем один раз в 1000 мс.

- Поэтому мы обернём вызов в декоратор: будем использовать **throttle(update, 1000)** как функцию, которая будет запускаться при каждом перемещении указателя вместо оригинальной update(). Декоратор будет вызываться часто, но передавать вызов в update() максимум раз в 1000 мс.

= Визуально это будет выглядеть вот так:

1. Для первого движения указателя декорированный вариант сразу передаёт вызов в update. Это важно, т.к. пользователь сразу видит нашу реакцию на его перемещение.
2. Затем, когда указатель продолжает движение, в течение 1000 мс ничего не происходит. Декорированный вариант игнорирует вызовы.
3. По истечению 1000 мс происходит ещё один вызов update с последними координатами.
4. Затем, наконец, указатель где-то останавливается. Декорированный вариант ждёт, пока не истечёт 1000 мс, и затем вызывает update с последними координатами. В итоге окончательные координаты указателя тоже обработаны. 

```javascript
function f(a) {
  console.log(a)
}

// f1000 передаёт вызовы f максимум раз в 1000 мс
let f1000 = throttle(f, 1000);

f1000(1); // показывает 1
f1000(2); // (ограничение, 1000 мс ещё нет)
f1000(3); // (ограничение, 1000 мс ещё нет)

// когда 1000 мс истекли ...
// ...выводим 3, промежуточное значение 2 было проигнорировано
``` 
- P.S. Аргументы и контекст this, переданные в f1000, должны быть переданы в оригинальную f.

- Решение 
```javascript
function throttle(func, ms) {

  let isThrottled = false,
    savedArgs,
    savedThis;

  function wrapper() {

    if (isThrottled) { // (2)
      savedArgs = arguments;
      savedThis = this;
      return;
    }

    func.apply(this, arguments); // (1)

    isThrottled = true;

    setTimeout(function() {
      isThrottled = false; // (3)
      if (savedArgs) {
        wrapper.apply(savedThis, savedArgs);
        savedArgs = savedThis = null;
      }
    }, ms);
  }

  return wrapper;
}
```

- ызов throttle(func, ms) возвращает wrapper.

1. Во время первого вызова обёртка просто вызывает func и устанавливает состояние задержки (isThrottled = true).
2. В этом состоянии все вызовы запоминаются в saveArgs / saveThis. Обратите внимание, что контекст и аргументы одинаково важны и должны быть запомнены. Они нам нужны для того, чтобы воспроизвести вызов позднее.
3. Затем по прошествии ms миллисекунд срабатывает setTimeout. Состояние задержки сбрасывается (isThrottled = false). И если мы проигнорировали вызовы, то «обёртка» выполняется с последними запомненными аргументами и контекстом.
- На третьем шаге выполняется не func, а wrapper, потому что нам нужно не только выполнить func, но и ещё раз установить состояние задержки и таймаут для его сброса. 

- **Декоратор debounce**
- Результат декоратора debounce(f, ms) – это обёртка, которая откладывает вызовы f, пока не пройдёт ms миллисекунд бездействия (без вызовов, «cooldown period»), а затем вызывает f один раз с последними аргументами.

- Другими словами, debounce – это так называемый секретарь, который принимает «телефонные звонки», и ждёт, пока не пройдет ms миллисекунд тишины. И только после этого передает «начальнику» информацию о последнем звонке (вызывает непосредственно f).

- Например, у нас была функция f и мы заменили её на f = debounce(f, 1000).

- Затем, если обёрнутая функция вызывается в 0, 200 и 500 мс, а потом вызовов нет, то фактическая f будет вызвана только один раз, в 1500 мс. То есть: по истечению 1000 мс от последнего вызова.

- …И она получит аргументы самого последнего вызова, остальные вызовы игнорируются.

- Ниже код этого примера (используется декоратор debounce из библиотеки Lodash): 
```javascript
let f = _.debounce(alert, 1000);

f("a");
setTimeout( () => f("b"), 200);
setTimeout( () => f("c"), 500);

// Обёрнутая в debounce функция ждёт 1000 мс после последнего вызова, а затем запускает: alert("c")
```
- Теперь практический пример. Предположим, пользователь набирает какой-то текст, и мы хотим отправить запрос на сервер, когда ввод этого текста будет завершён.

- Нет смысла отправлять запрос для каждого набранного символа. Вместо этого мы хотели бы подождать, а затем обработать весь результат.

- В браузере мы можем настроить обработчик событий – функцию, которая вызывается при каждом изменении поля для ввода. Обычно обработчик событий вызывается очень часто, для каждого набранного символа. Но если мы воспользуемся debounce на 1000мс, то он будет вызван только один раз, через 1000мс после последнего ввода символа.

- В этом живом примере обработчик помещает результат в поле ниже, попробуйте:

- Видите? На втором поле вызывается функция, обёрнутая в debounce, поэтому его содержимое обрабатывается через 1000мс с момента последнего ввода.

- Таким образом, debounce – это отличный способ обработать последовательность событий: будь то последовательность нажатий клавиш, движений мыши или ещё что-либо.

- Он ждёт заданное время после последнего вызова, а затем запускает свою функцию, которая может обработать результат.

- **Задача — реализовать декоратор debounce.**

- Подсказка: это всего лишь несколько строк, если вдуматься:

```javascript
function debounce(func, ms) {
  let timeout;
  return function() {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, arguments), ms);
  };
}
```

- Вызов debounce возвращает обёртку. При вызове он планирует вызов исходной функции через указанное количество ms и отменяет предыдущий такой тайм-аут. 

