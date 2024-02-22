# Вопросы и ответы на фроненд разработчика

### HTML. Cемантическая верстка. Web Accessibility
- **Cемантическая верстка. Зачем нужна**
- Семантическая вёрстка — подход к разметке, который опирается не на содержание сайта, а на смысловое предназначение каждого блока и логическую структуру документа. Даже в этой статье есть заголовки разных уровней — это помогает читателю выстроить в голове структуру документа. Так и на странице сайта — только читатели будут немного другими. 

- Зрячие пользователи могут без проблем с первого взгляда понять, где какая часть страницы находится — где заголовок, списки или изображения. Для незрячих или частично незрячих всё сложнее. Основной инструмент для просмотра сайтов не браузер, который отрисовывает страницу, а скринридер, который читает текст со страницы вслух.

- Этот инструмент «зачитывает» содержимое страницы, и семантическая структура помогает ему лучше определять, какой сейчас блок, а пользователю понимать, о чём идёт речь. Таким образом семантическая разметка помогает б_о_льшему количеству пользователей работать с вашим сайтом. Например, наличие заголовков помогает незрячим в навигации по странице. У скринридеров есть функция навигации по заголовкам, что ускоряет знакомство с информацией на сайте.

= Чтобы сайт был выше в поисковиках. Поисковики не разглашают правила ранжирования, но известно, что наличие семантической разметки страниц помогает поисковым ботам лучше понимать, что находится на странице, и в зависимости от этого ранжировать сайты в поисковой выдаче. 

- **Доступность (Accessibility**, A11y в англоязычной среде) в веб-разработке — это обеспечение возможности использования сайтов как можно большим числом людей, включая тех, чьи способности как-либо ограничены.

- Технологии облегчают жизнь многим людям. А людям с ограниченными возможностями технологии дают такие возможности, которые ранее им были недоступны. Доступность в контексте разработки подразумевает создание такого контента, пользоваться которым мог бы каждый, несмотря на индивидуальные физические или когнитивные способности и вне зависимости от того, как они получают доступ в сеть.


### CSS. Вес селектора. CSS-архитектура (БЭМ, OOCSS, SMACSS)
- **Вес селектора**
- Все CSS-селекторы имеют свой вес, который определяет как взаимодействуют одинаковые свойства, заданные в разных местах кода одному и тому же элементу.
- Иногда это может создавать трудности, когда свойство, объявленное ниже в коде, перекрывается тем, что объявленно выше, потому что селектор первого более специфичен (имеет больший вес).

- Вот пример проблемы. Есть див с id="container", внутри него некоторый текст и список ссылок.

```html
<div id="container">
  <p><a href="#">link in P</a></p>

  <ul class="list">
    <li><a href="#">Link1</a></li>
    <li><a href="#">Link2</a></li>
  </ul>
</div>
```
- Сначала задаём всем ссылкам внутри #container оранжевый фон:

```css
#container A {
  background: orange;
}
``` 
- А потом, чтобы в списке .list внутри контейнера ссылки имели зелёный фон, ниже дописываем такое:
```css
.list A {
  background: mediumspringgreen;
}
``` 

- Казалось бы, ссылки в тексте должны получить оранжевый фон, а ссылки в списке — зеленый, но нет 

- Почему так? Потому, что первый селектор содержит ID и перевешивает второй, то есть:
```css
#container A > .list A 
```
**Вес селектора**
- Специфичность селектора рассчитывается по 4-м позициям:
#container A
1. style=""
2. #id 
3. .class
4. <tag>

#container UL A
1. style=""
2. #id 
3. .class
4. <tag>

**Вес селекторов (по убыванию):**
```css
style="" 1,0,0,0

#id 0,1,0,0

.class 0,0,1,0

[attr=value] 0,0,1,0

LI 0,0,0,1

* 0,0,0,0 
``` 

- У стилей, заданных в атрибуте style, на первой позиции будет единица — 1,0,0,0. Это самая высокая специфичность, которая перевешивает свойства, заданные другими способами.

- Переопределить стили, заданные в style, можно дописав !important к значению свойства в таблице стилей.

- Обратный вариант — универсальный селектор *, он не имеет веса: 0,0,0,0. 

- Примеры:
```css
LI 0,0,0,1 — селектор по тегу

UL LI 0,0,0,2 — селектор c двумя тегами весит больше, чем с одним.

.orange 0,0,1,0 — селектор с классом весит больше, чем селектор с тегом.

.orange A SPAN 0,0,1,2 — селектор перевесит предыдущий, потому что помимо класса содержит два тега.

#page .orange 0,1,1,0 — селектор с ID перевесит всё, кроме inline-стилей.

Теперь сравним селекторы из исходного примера:

#container A 0,1,0,1

.list A 0,0,1,1

0,1,0,1 > 0,0,1,1 — хорошо видно, что селектор с ID весит больше, чем селектор с классом, поэтому все ссылки имеют оранжевый фон, хотя ниже в коде им задан зеленый. 
``` 

- Варианты решений

1. Добавить !important

```css
#container A {
  background: orange;
}

.list A {
  background: mediumspringgreen !important;
}
```
- Ссылки получат зеленый фон, быстро и легко. Но это плохой способ, потому что код запутывается ещё больше. Со временем для переопределения !important в одном месте может потребоваться добавить его в других местах. Иерархичность начнет работать не сверху низ и от общего к частному, а как попало. В конце-концов поддерживать такой код будет весьма проблематично.

- В общих случаях использовать !important не рекомендуется, но может пригодиться, если нужно, чтобы часто используемый блок на всех страницах выглядел одинаково, независимо от окружения. В любом случае нужно всегда четко понимать зачем вы его используете.

2. Следующий очевидный способ — добавить #container ко второму селектору, чтобы увеличить его вес:
```css
#container A {
  background: orange;
}

#container .list A {
  background: mediumspringgreen;
}
``` 

- Это тоже сработает, но решение так себе: удлиняется цепочка селекторов (что может отразиться на скорости отрисовки страницы) и ухудшается читаемость кода. Так тоже делать не стоит.

- 1-й и 2-й способ могут использоваться, если у вас нет доступа к разметке, а в ней нет нужных классов. Если же вы можете редактировать разметку либо классы у элементов таки есть — используйте последний способ, самый правильный:

3. **Просто не используйте в стилях селекторы с ID, используйте классы.** 

- Посмотрим на разницу между #container и с .container:
```css
#container A 0,1,0,1 — селектор с ID перевешивает всё вне зависимости от своего расположения в коде.
```
- Заменим в разметке страницы id на class:
```css
.container A0,0,1,1 — селектор с классом весит меньше, он менее специфичен.
```
- Селектор ссылок в списке весит столько же:
```css
.list A 0,0,1,1
```

- Это означает, что при равном весе селекторов применятся стили, объявленные ниже в коде. То есть достаточно будет просто написать стили, следуя от общего к частному, сверху вниз.

В итоге разметка может быть такой:
```css
<div class="container">
  <p><a href="#">link in P</a></p>

  <ul class="list">
    <li><a href="#">Link1</a></li>
    <li><a href="#">Link2</a></li>
  </ul>
</div>
``` 

- А стили — такими:
```css
.container A {
  background: orange;
}

.list A {
  background: mediumspringgreen;
}
```
И код работает так, как ожидается

- Если id в вашей разметке уже используется в Js, логичнее будет добавить элементу класс и перевесить стили на него. Если же id участвует только в разметке — лучше заменить его на class.

- **В качестве общих рекомендаций так же следует упомянуть, что нужно как можно меньше использовать селекторы по тегу и как можно больше — селекторы по классу. Это поможет избежать проблем при повторном использовании блоков сайта, а при использовании "умных" классов — может значительно сократить цепочки селекторов, увеличить читабельность кода и скорость отрисовки страницы.**
