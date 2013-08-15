# classList API

Честно говоря, чувствую себя жуликом потому что пишу о JavaScript для HTML5 
Doctor. Я бы чувствовал себя жуликом даже если бы писал о JavaScript для 
захламленной рекламой контент-фермы, что уж говорить об HTML5 Doctor.

Дело в том, что речь пойдет об удивительно простом `classList` API. Если вы не 
слишком хорошо владеете Javascript-фу и опасаетесь HTML5 API, этот API идеально 
подходит для начала и вы приятно удивитесь насколько просто его использовать.

Название `classList` API говорит само за себя. Он получает список классов для 
элемента HTML и позволяет управлять им с помощью JavaScript.

До появления этого сокровища в HTML5 работа с классами была сплошной тягомотиной, 
но то что однажды было извилистой заросшей тропкой в дебрях, кишащих волками, 
сейчас стало залитой солнцем дорожкой, на которой хоть на роликах катайся. 

## Извлекаем `classList`

Извлечь `classList` можно очень просто используя `element.classList` вот так:

    <p id="bad-joke" class="oh my giddy aunt">Две собаки гуляют в парке. Одна говорит: «Какой чудный день, вы не находите?» Вторая отвечает: «Чёрт побери, говорящая собака!»</p>

    <script>
        var joke = document.getElementById('bad-joke');
        console.log(joke.classList);
    </script>

Этот код возвратит нечто вроде `["oh", "my", "giddy", "aunt"]`. Оформление 
выведенного результата будет отличаться в зависимости от браузера, однако по 
сути мы получим объект с списком классов для элемента.

Если точнее, это объект, который преобразуется в значение атрибута класса для рассматриваемого элемента. [Попробуйте сами с открытой консолью][1].

    {
        0: "oh",
        1: "my",
        2: "giddy",
        3: "aunt"
    }

Для `classList` нет спецификации как таковой ([о нём есть предложение в 
спецификации DOM4][2]), однако есть спецификация для `DOMTokenList`, который 
является типом `classList`, так что за информацией о работе со списками классов 
обратитесь к [спецификации `DOMTokenList`][3].

## Тип `classList`

Если вам нужно подтвердить тип `classList`, используйте 
`element.classList instanceof DOMTokenList`, что возвратит `true`. 
`typeof element.classList` возвращает `object`, что не слишком-то информативно 
если учитывать что в JavaScript всё является объектом.

## Использование списка классов

В спецификации `DOMTokenList` перечислен ряд методов, которые могут быть 
использованы для `classList`:

* `add()` для добавления класса в список
* `remove()` для удаления класса из списка
* `contains()` для проверки наличия класса в списке
* `toggle()` для переключения класса — добавления при отсутствии в списке и 
удаления при наличии (с некоторыми особенностями)
* `item()` для возвращения класса в определённой позиции в списке
* `toString()` для превращения списка в строку
* `length` для возвращения количества классов в списке
*  `value` для добавления дополнительных свойств и методов для объекта 
`classList`

## `classList.add()`

Более лёгкий способ добавить класс для элемента и не придумаешь. Просто 
прописываем нужный класс в качестве аргумента для метода `add()`.

    joke.classList.add('beryl');

[Откройте демо][4] и посмотрите на открывающий тег `<p>` в вашем любимом 
веб-инспекторе до и после нажатия кнопки и вы увидите что он меняется с 
`<p id="bad-joke" class="oh my giddy aunt">` на 
`<p id="bad-joke" class="oh my giddy aunt beryl">`.

Класс в `<p>` добавлен с помощью всего лишь этой единственной строки. Нет нужды 
вставлять встроенные стили или использовать другие грубые приёмы в этом роде.

Соответствующий объект `classList`:

    {
        0: "oh",
        1: "my",
        2: "giddy",
        3: "aunt",
        4: "beryl"
    }

## `classList.remove()`

Удалить класс так же просто. `joke.classList.remove('beryl')` возвращает нас к 
исходному примеру.

[Посмотреть демо][5]

## Добавление и удаление нескольких классов

В спецификации `DOMTokenList` говорится о «маркерах» *(англ. «tokens» — прим. 
переводчика)*, в описании методов `add()` и `remove()` они упомянуты в 
множественном числе.

> Метод add(tokens…) […] Если один из *маркеров* является пустой строкой […] 
Если один из *маркеров* содержит любой неотображаемый символ в системе ASCII […] 
Для каждого *маркера* из *маркеров* — [Спецификация DOM][6]

<s>Пока ни в одном браузере не реализован метод добавления/удаления более чем 
одного класса сразу, но для нас не составит труда дополнить прототип 
`DOMTokenList` с помощью написанной вручную функции:</s>

В браузерах на движке Blink можно добавлять и удалять несколько классов 
одновременно с помощью `element.classList.add('oh','my')`, кроме того не 
составит труда дополнить прототип `DOMTokenList` с помощью написанной вручную 
функции пока поддержка не будет внедрена во всех браузерах:

Спасибо Девиду и Кэлли за то что подметили это в комментариях. 

    DOMTokenList.prototype.addmany = function(classes) {
        var classes = classes.split(' '),
            i = 0,
            ii = classes.length;
    
        for(i; i<ii; i++) {
            this.add(classes[i]);
        }
    }

Так же и для удаления:

    DOMTokenList.prototype.removemany = function(classes) {
        var classes = classes.split(' '),
            i = 0,
            ii = classes.length;
    
        for(i; i<ii; i++) {
            this.remove(classes[i]);
        }
    }

[Вот демо в котором можно добавить и удалить несколько классов одновременно.][7]

В 2011 году была подана жалоба (касательно документа, который уже не существует) 
с пожеланием разрешить для `classList.add()` и `classList.remove()` список с 
пробелом в качестве разделителя или массив.

«Маркеры» в множественном числе наталкивают на мысль что массив должен быть 
разрешен, но на практике он не работает. Указание выводить 
`InvalidCharacterError` если в «маркерах» присутствуют пробелы все еще в силе, 
так что такой подход точно не сработает.

Если вы хотите заменить весь `classList` полностью иным набором классов, можете 
использовать эти две функции.

## `classList.contains()` 

Этот метод при проверке наличия класса в списке возвращает булевы `true` или 
`false`.

Пример:

    joke.classList.contains('aunt') === true;
    joke.classList.contains('uncle') === false;

`contains()` полезен если вам нужно выполнить проверку на наличие класса в 
списке перед выполнением действия, которое зависит от присутствия этого класса 
(или его отсутствия).

    if(joke.classList.contains('beryl') {
        joke.classList.remove('beryl');
    }

Можно использовать `contains()` как дополнение для двух предыдущих функций 
удаления и добавления нескольких классов чтобы убедиться что они не будут 
пытаться добавить дубликат класса для элемента или удалить класс, который в 
списке отсутствует:

    DOMTokenList.prototype.addmany = function(classes) {
        var classes = classes.split(' '),
            i = 0,
            ii = classes.length;

        for(i; i<ii; i++) {
            if(!this.contains(classes[i])) {
                this.add(classes[i]);
            }
        }
    }

и:

    DOMTokenList.prototype.removemany = function(classes) {
        var classes = classes.split(' '),
            i = 0,
            ii = classes.length;

        for(i; i<ii; i++) {
            if(this.contains(classes[i])) {
                this.remove(classes[i]);
            }
        }
    }

## `classList.toggle()` 

В большинстве случаев `classList.toggle()` также довольно незамысловат. Как 
правило, действие программы или пользователя запускает функцию, которая удалит 
или добавит класс в зависимости от того присутствует он уже в списке или нет. 

Хорошим примером будет простое действие отображения/скрытия:

    button.addEventListener('click', function() { element.classList.toggle('is-visible'); }, false);

Взгляните на демо двухуровневого меню, которое занимает немного меньше места при 
небольшой области просмотра за счёт использования `classList.toggle()` в целях 
оптимизации подменю.

## `force`

И все же у `toggle()` есть небольшая хитрость. При необходимости он может 
принимать второй параметр `force`.

Если для `force` задано значение `true`, класс может быть добавлен, но не удалён. 
Если задано `false`, случится противоположное: класс сможет быть удалён, но не 
добавлен.

Теперь этот метод сильно напоминает `add()` and `remove()`, но есть существенное 
отличие: `toggle()` с `force` возвращает `true` когда класс добавлен и `false` 
когда он удалён, `add()` и `remove()` возвращают `undefined`.

В момент написания статьи параметр `force` различают только браузеры на движке 
Blink, так что откройте [демо с `true`][10] и [демо с `false`][11] в Opera 15, 
Opera для Android или в последней версии Chrome чтобы увидеть как это работает.

Браузеры, не поддерживающие `force`, полностью его игнорируют и охотно удаляют и 
добавляют классы в зависимости от их присутствия в списке без каких-либо 
исключений. 

Когда поддержка будет лучшей, для определения наличия или отсутствия класса в 
списке можно будет использовать немного более изящный синтаксис. Вместо:

    var foo = function() {
        joke.classList.add('beryl');
    }

    foo();

    if(joke.classList.contains('beryl')) {
        // выполнение действий
    }

можно написать:

    var foo = function() {
        return joke.classList.toggle('beryl', true);
    }

    if(foo()) {
        // выполнение действий
    }

## `classList.item()`

Метод `item()` в JavaScript довольно распространён, большей частью для объектов 
NodeList. Он возвращает значение позиции в `index` считая с нуля.

В нашем примере:

    <div id="bad-joke" class="oh my giddy aunt">
        <p>"Стук-стук."</p>
        <p>"Входите, открыто."</p>
    </div>

    <script>
        var joke = document.getElementById('bad-joke'),

            whats_item_0 = function() {
                return joke.classList.item(0);
            },

            whats_item_2 = function() {
                return joke.classList.item(2);
            };

        console.log(whats_item_0()); // выводит "oh"
        console.log(whats_item_2()); // выводит "giddy"
    </script>

[Посмотреть демо.][12]

Для `classList.item()` нельзя использовать присваивание, потому 
`joke.classList.item(3) = 'uncle'` выдаст ошибку.

Если вы надеетесь с помощью `classList.item()` присвоить значение элементу в 
определённой позиции в списку, боюсь вы будете разочарованы. Нет такого метода 
`DOMTokenList` который давал бы над списком контроль такого рода.

## `classList.toString()`

Если вам однажды понадобится превратить `classList` в строку, используйте этот 
метод. `toString()` — это еще один метод JavaScript, характерный не только для 
`DOMTokenList`.

Спецификация W3C по этому поводу говорит только что «объекты DOMTokenList должны 
быть преобразованы в строку, которая лежит в его основе». WHATWG остановилась на 
определении «Обработчик, преобразовывающий список в строку, должен возвратить 
результат преобразования для соответствующего списка маркеров».

Разница только в формулировке.

Он не принимает никакие параметры когда используется с `classList` и возвращает 
все классы для элемента в виде строки с пробелом-разделителем.

    joke.classList.toString();
    // возвращает «oh my giddy aunt»;

## `classList.length`

`length` также является встроенным методом JavaScript. Он возвращает количество 
символов в строке, количество позиций в массиве, количество аргументов в функции 
или (в нашем случае) количество классов в `classList`. В нашем примере «oh my 
giddy aunt» оно даст `4`.

    joke.classList.length
    // возвращает 4

Просто и понятно, я надеюсь.

## `classList.value`

Сами по себе методы `item()`, `toString()`, и `length` не слишком полезны. 
Однако их можно использовать в сочетании друг с другом и с остальными методами 
`DOMTokenList` чтобы решить некоторые проблемы, о которых я упоминал ранее. 

Помните что `classList` является обычным объектом JavaScript, следовательно для 
него можно назначать свойства так же как и для любого другого объекта:

    classList.use_in_production = false;
    classList.id = 5;
    classList.roof = 'thatch';

И, конечно же, можно использовать методы. Вот как с помощью `length`, 
`toString()`, `add()`, и `remove()` целый список можно заменить новым:

    element.classList.replace = function(classes) {
        var i = 0,
            ii = this.length,
            old_string = this.toString(),
            old_array = old_string.split(' '),
            new_array = classes.split(' '),
            j = 0,
            jj = new_array.length;

        // удаляем все классы
        for(i; i<ii; i++) {
            this.remove(old_array[i]);
        }

        // добавляем новые
        for(j; j<jj; j++) {
            this.add(new_array[j]);
        }
    };

Также можно добавить класс в любой позиции в списке. Для этого используется 
приём с `contains()`, `item()`, `remove()`, `toString()`, и `replace()`, который 
мы только что рассмотрели:

    element.classList.insert = function(insert,position) {
        // проверяем присутствует ли класс в classList
        if(this.contains(insert)) {
            if(this.item(position) === insert) {
                // если он уже добавлен в правильной позиции, продолжать нет нужды
                return;
            } else {
                // удаляем его, он расположен не там где нужно
                this.remove(insert);
            }
        }

        var classes = this.toString(),
            classes_array = classes.split(' ');

            classes_array.splice(position, 0, insert);

            new_list = classes_array.join(' ');

            // заменяем текущий classList
            this.replace(new_list);
    };

Если смотреть на вещи реалистично, такие приёмы скорее всего стоит использовать 
с целью расширения возможностей `DOMTokenList` как в случае с `addmany()` и 
`removemany()`, но если вам нужно нечто более узконаправленное для конкретного 
`classList`, с которым вы работаете, можно использовать `value`.

## Поддержка браузерами и полифилы

`classList` API [работает практически во всех современных версиях браузеров][13]. 
По меньшей мере базовая поддержка была реализована начиная с Firefox 3.6, Opera 
11.50, Chrome 8 и Safari 5.1.

Белым пятном являются IE9 и старше и [всё еще довольно популярный Android 2.3][14] 
и старше.

[Существует по крайней мере два полифила][15], которые помогут справиться с 
этими пятнами. 

Написанный на скорую руку [полифил Девона Говета (Devon Govett)][16] 
обеспечивает поддержку только для IE9, также стоит почитать комментарии, 
содержащие неплохую информацию кросбраузерном JavaScript.

[Полифил Эли Грея (Eli Grey)][17] дает более широкую поддержку. Он работает для 
IE8 и обеспечивает базовую поддержку `classList.add()`, `classList.remove()` и 
`classList.toggle()` для Android 2.1 и младше. 

Также есть [полифил для поддержки `force`][18] от *Егора Халимоненко* под 
браузеры, которые поддерживают `classList`, но не понимают `force` для `toggle()`.

Если вам нужна только простая проверка возможностей, можете использовать:

    if('classList' in document.createElement('a')) {
        // используем classList API
    }

## Итог

Можно рассматривать `classList` API в двух плоскостях. В большинстве случаев это 
простой способ добавить, удалить и проверить наличие отдельных классов для 
элемента HTML. На данный момент я использую его в каждом проекте и мне очень 
редко приходится отклоняться от этих простых методов. Не могу придумать в каком 
случае `item()`, `length` или `toString()` могли бы пригодиться сами по себе, 
поэтому они меня не особо волнуют.

С другой стороны, здесь вся соль в работе с объектом, поиске функциональных 
способов управлять состоянием веб-страниц, разделении функций, прогрессивном 
улучшении. 

Только от вас зависит для чего вы будете его использовать, но как я говорил в 
начале, с `classList` API легко разобраться и он делает легким то, что раньше 
было трудным. Советую прислушаться к тому, о чём я здесь написал и 
поэкспериментировать в своих собственных проектах если для вас `classList` API 
является новым понятием; если же он вам уже хорошо знаком, поделитесь в 
комментариях что можно было бы рассказать лучше. 

Спасибо!

[1]: http://html5doctor.com/demos/classlist/classlist.html
[2]: http://www.w3.org/TR/dom/#dom-element-classlist
[3]: http://dom.spec.whatwg.org/#domtokenlist
[4]: http://html5doctor.com/demos/classlist/classlist.add.html
[5]: http://html5doctor.com/demos/classlist/classlist.remove.html
[6]: http://dom.spec.whatwg.org/#dom-domtokenlist-add
[7]: http://html5doctor.com/demos/classlist/classlist.add-remove-many.html
[8]: http://lists.w3.org/Archives/Public/public-html-bugzilla/2011Sep/0032.html
[9]: http://html5doctor.com/demos/classlist/classlist.toggle.html
[10]: http://html5doctor.com/demos/classlist/classlist.toggle-true.html
[11]: http://html5doctor.com/demos/classlist/classlist.toggle-false.html
[12]: http://html5doctor.com/demos/classlist/classlist.item.html
[13]: http://caniuse.com/#search=classlist
[14]: http://developer.android.com/about/dashboards/index.html
[15]: https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills#classlist
[16]: https://gist.github.com/devongovett/1381839
[17]: https://github.com/eligrey/classList.js
[18]: https://gist.github.com/termi/3952026