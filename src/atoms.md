# Работа с изменяемым состоянием

Практически любое приложение меняет своё состояние в процессе работы. В JavaScript состояние хранится в глобальных переменных, в лексических замыканиях, в атрибутах объектов. ClojureScript, как hosted язык, тоже может создавать JS­объекты и изменять их содержимое в процессе работы программы. Однако, в мире JavaScript достаточно часто эти объекты, изображающие фрагменты одного большого "состояния всей системы", изменяются разными частями программы и уследить за всеми изменениями бывает достаточно сложно, если вообще возможно. Поэтому в Clojure используются неизменяемые структуры данных, доступ к которым осуществляется через *изменяемые ссылки*.

## Атомы

В Clojure существует несколько видов изменяемых ссылок, обладающих различными свойствами, однако в ClojureScript доступен ровно один вид: *атомы*. В процессе работы с атомом мы получаем текущее значение по ссылке, создаём новое значение на основе старого и только после этого перенаправляем ссылку на это новое значение. При этом непосредственно изменение ссылки происходит мгновенно, насколько сильно бы мы не меняли само значение. Поэтому не возникает печально известное *состояние гонки*, когда часть данных уже поменялась, а часть ещё не успела измениться и какие-то другие процессы в программе могут получить доступ к *неконсистентному состоянию*. При использовании атомов по ссылке всегда доступно либо старое состояние, либо уже новое, никакой возможности получить частично изменённые данные нет!

Создаётся атом функцией `(atom начальное-значение)` и обычно связывается с некоторым именем:

```clojure
(def counter (atom 0))
```

Получить текущее значение, на которое ссылается атом, можно с помощью так называемого *разыменования* (*dereferencing*), указав перед именем символ `@`: `(println @counter)`. При разыменовывании вы получаете именно текущее значение, поэтому булево выражение `(= @counter @counter)` иногда может оказаться ложным — если другая часть программы успеет изменить ссылку между двумя разыменованиями.

И, наконец, перенаправить ссылку на новое значение можно с помощью функции `(reset! имя-атома новое-значение)`.

> Заметьте, функция `set!`, изменяющая значение атрибутов JS­объектов, и функция `reset!`, перенаправляющая ссылки атомов, имеют восклицательные знаки в именах. Так в Clojure принято называть функции, за использованием которых нужно внимательно следить. Как правило, это функции, меняющие что-то за пределами видимости текущего участка кода.

## `reset!` против `swap!`

У функции `reset!` есть один недостаток: если вы разыменовали атом, долго вычисляли новое значение, а потом перенаправили на него атом с помощью `reset!`, то может так случиться, что какой-то другой процесс успеет изменить атом после вашего разыменования, но до вашего `reset!`. Если такое случится, то часть информации будет потеряна. Вот так эта ситуация может выглядеть:

```clojure
;; [ первый процесс ]                | ;; [ второй процесс ]
;;                                   | ;;
;; получаем текущий баланс счёта     | ;;
(let [acc @bob-account]              | ;;
  ;; что-то долго делаем             | (let [acc @bob-account]
  ;; ...                             |   ;; быстро снимаем 10 кредитов
  ;; ...                             |   (reset! bob-account (- acc 10)))
  ;; списываем 100 кредитов          | ;;
  (reset! bob-account (- acc 100)))  | ;;
```

Здесь первый процесс даже не узнает о том, что баланс счёта уже поменялся на -10 кредитов, и перезапишет с расходом -100 кредитов, а первые -10 потеряются!

Чтобы попадать в такие ситуации пореже, обычно стараются разыменовывать атомы максимально близко к вызову `reset!`. Однако есть более безопасный способ: функция `swap!`. Эта функция не принимает новое значение для атома, а вместо этого ожидает, чтобы вы предоставили ей *функцию из старого значения в новое*. При этом само внутреннее устройство атома гарантирует, что при вызове такой *функции­модификатора* между получением старого значения и переключением ссылки на новое никакой другой процесс не вклинится. То есть использование `swap!` делает изменение истинно *атомарным* (именно поэтому атомы называются так как называются).

Чтобы списать со счёта Боба десять кредитов, нужно применить `swap!` таким образом: `(swap! bob-account - 10)` — значение по ссылке будет передано в функцию­модификатор первым, но вы можете указать и другие аргументы для функции. Таким образов в одной части программы можно сделать `(swap! bob-account - 100)`, а в другой выполнить `(swap! bob-account - 10)`, и общие траты составят -110!

## Атомарность и побочные эффекты

Функция `swap!` устроена так, что при перед вызовом функции­модификатора запоминает старый адресат ссылки. А после того, как модификатор вернул новое значение, то `swap!` проверяет, указывает ли ссылка на запомненный ранее адрес. Если ссылка оказалась перенаправлена, пока модификатор выполнял вычисления, то `swap!` запускает модификатор ещё раз — уже с новым значением (тем, на которое теперь указывает ссылка) в качестве аргумента.

Такие повторы могут происходить не единожды, если с атомом одновременно и достаточно активно работают несколько процессов в рамках одной программы. Поэтому очень важно не производить в теле функции­модификатора никакие побочные эффекты вроде изменения других атомов, отправки запросов к серверу и тому подобные действия. Иначе вы рискуете ощутить эти побочные эффекты несколько раз, даже если сделали один вызов `swap!`.

Рекомендация тут может быть такая: вам нужно выносить любые затратные с точки зрения времени или сопряжённые с побочными эффектами вычисления за пределы функции­модификатора, а `swap!` использовать только в самом конце. Следовать этой рекомендации обычно не слишком сложно, и отделение побочных эффектов от изменения состояния идёт на пользу читаемости кода.
