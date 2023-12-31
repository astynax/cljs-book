# Синтаксис Clojure

## Read-Eval-Print Loop

Пока мы будем рассматривать EDN и основы синтаксиса Clojure, вам стоит держать под рукой Clojure REPL. "*REPL*" — сокращение от "Read-Eval-Print Loop". Это такая программа, которая позволяет вводить выражения на языке программирования, вычисляет их, печатает результат и ждёт следующей команды. У вас, скорее всего, уже имеется JavaScript REPL в вашем Web Browser, только называется он "Консоль разработчика" или как-то похоже.

Clojure — язык компилируемый, поэтому ему нужен компилятор, который пока в браузеры не встроили. Но на начальном этапе можно воспользоваться сайтом https://tryclojure.org — это как раз и есть "Clojure REPL в браузере", реализованный с помощью ClojureScript. Можете попробовать ввести на сайте строчку `(println "Hello world!")` и нажать <Enter>, в ответы вы должны получить соответствующее сообщение.

## Числа, строки и прочие примитивы

Числа в Clojure записываются так же, как в большинстве других языков: `42`, `3.14` — это целое число и число с плавающей точкой. Строки записываются в двойных кавычках: `"Hello"`.

Для булевых значений в языке имеются отдельные литералы: `true` и `false`, при этом булев в Clojure самостоятельный, а не является фасадом для чисел "1" и "0", как бывает в некоторых языках.

Отдельно стоит отметить литерал, отвечающий за "отсутствие значения", пусть даже сам по себе он значением является. Это литерал `nil`, по смыслу похожий на `null` в JavaScript. Если вы пробовали выполнить в REPL кусочек кода, упомянутый в предыдущем разделе, то вы видели вывод `nil` после приветствия: `nil` является результатом вычисления выражения `(println ...)`, потому что именно это значение вернула функция `println`.

## Ключевые слова

*Ключевые слова* (*keywords*) — это имена, начинающиеся с двоеточия, и включающие в себя буквы, цифры и некоторые другие символы, такие как знак минуса. Ключевые слова ценны тем, что всегда представляют только самих себя, ни больше, ни меньше. Ключевое слово, сконструированное с помощью литерала или другим способом, всегда будет означать то же самое, что означают другие "экземпляры" этого ключевого слова когда бы и где бы вы их не создали. Поэтому keywords часто используются для описания каких-то *номинальных значений* вроде `:carrot` или `:city`. Вот ещё несколько примеров:

- `:kebab-cased-name`
- `:0xFF00FF` — нет, это не число!
- `:42` — и это не число
- `:/usr/local/bin/gzdoom` — а это не путь к файлу

Попробуйте ввести каждый из этих символов в REPL и опробуйте заодно несколько своих keywords.

Вы можете спросить, а зачем нужен отдельный тип для "просто имён". В других языках в роли таких "просто каких-то имён" часто используются строки. Вот только у этого решения есть недостатки. Так, две строки, созданные в разных частях программы, могут быть равны посимвольно, но чтобы удостовериться, что вы видите два экземпляра одной и той же сущности, вам придётся произвести это самое посимвольное сравнение. А ещё использование строк не только в роли текста, но и в роли "имён общего назначения" заставляет каждый раз, видя строковый литерал, думать, смотрите ли вы на текст или на другого рода значение — именно поэтому во многих руководствах по оформлению кода рекомендуется строковые константы не использовать как литералы, но сохранять предварительно в переменные (константные) и ссылаться на них по именам переменных.

Keywords, в отличие от "строк как имён", не являются текстом, а значит нет никаких проблем с двойственностью "текст VS имя". А ещё ключевые слова *быстро сравниваются* друг с другом: среда исполнения помнит все встреченные в коде ключевые слова и сравнивает их адреса в памяти вместо посимвольного сравнения имен.

## Символы

*Символы* (*symbols*) похожи на ключевые слова, только они не предваряются двоеточием. И символы тоже что-то именуют. Но то, что в данном конкретном месте кода означает символ, всецело* зависит от контекста*: в одном месте имя может означать функцию, в другом строку, в третьем может не означать ничего, то есть будет *не определено*. Можете пока думать о символах, как об именах переменных и функций, потому что чаще всего символы именно в качестве таковых и используются.

Важно понять, что при вычислении выражений символ должен быть связан с чем-то, иначе вы получите ошибку "Could not resolve symbol: XYZ". Чтобы использовать символ сам по себе, нужно его *процитировать* (*quote*), то есть добавить перед именем символ `'` (одинарная кавычка): так вы сообщите вычислителю, что вам важно само *имя символа*, а не значение, которое сейчас с именем связано. Вот несколько примеров символов, каждый из которых цитирован и может быть введён в REPL, где будет вычислен сам в себя:

- `'hello`
- `'/bin/bash`

## Векторы и отображения

*Векторы* выполняют роль массивов из других языков — это упорядоченные наборы значений. К элементам векторов можно обращаться по индексу. Вот несколько примеров векторов:

- `[]` — пустой вектор
- `[1 2 3]` — вектор чисел
- `[true false 3.14 nil ["a" 7]]` — вектор с элементами разных типов

*Отображения* (*maps*) выполняют роль хранилищ значений, доступных по ключу. Подобные структуры в других языках называются *словарями* или *хеш-таблицами*, а в JavaScript это объекты. Вот несколько примеров отображений:

- `{}` — пустое отображение
- `{"a" 1 "b" 2}` — отображение, в котором по ключу `"a"` хранится значение `1`, а по ключу `"b"` значение `2`
- `{1 "a" 2 "b"}` — а здесь уже ключами выступают числа, а значениями являются строки

Заметьте, ни в векторах ни в отображениях *не используются запятые* для разделения элементов, а в отображениях даже ключи со значениями не связаны в пары никакими конструкциями, они просто идут друг за другом через одного. Единственный разделитель здесь, это пробел. И переносить на новую строку длинный литерал можно в любом месте, где имеется этот пробел. Да, к такому нужно привыкнуть. Но в целом такое *единообразие* записи литералов, как "разделённых через пробелы последовательностей чего-то либо", позволяет меньше заботиться о синтаксисе и больше думать о сути.

Выше в примерах вы видели строки и числа в роли ключей, однако гораздо чаще вы будете встречать в роли ключей ключевые слова. Ведь что есть ключ отображения, как не "имя значения"? И как значения, ключевые слова тоже используются частенько. Вот вам пример структуры, описывающей группу домашних животных:

``` clojure
[{:type :pet  ; <- тут и ключ, и значение это keywords
  :kind :dog  ; (к слову, с точки с запятой начинаются комментарии)
  :name "Spike"
  :age 3}
 {:type :pet
  :kind :cat
  :name "Thomas"
  :age 2}]
```

## Списки

У списков в Clojure роль особая. Как структура данных, односвязные списки подходят для не слишком широкого круга задач, поэтому именно в "данных" списки встретить можно довольно редко. А вот в "коде" списки превалируют: почти всегда, когда вы видите список, подразумевается *вызов функции* или другая *управляющая конструкция*, большая часть из которых — макросы, то есть, опять же, функции, только работающие с кодом. Вычислитель вообще всегда пытается трактовать литерал списка как вызов функции, если вы список предварительно не процитируете!

Вызов функции всегда выглядит как список, первый элемент которого вычисляется в функцию, а потом этой функции передаются остальные элементы списка в качестве аргументов. Чаще всего первым элементом будет выступать символ, который на момент вычисления связан со значением-функцией. Вот несколько примеров, опробуйте их в REPL:

- `(println "Hi!")`
- `(+ 1 2)`
- `(if (> 2 1) :greater :not-greater)`

Заметьте, как выглядят сложение и сравнение чисел: оператор идёт перед операндами! Дело в том, что в лиспоподобных языках обычно нет такой вещи как "оператор", есть только функции, связанные с символами `+`, `>` и так далее. А поскольку имя функции всегда идёт первым в списке, то `+` в выражении, вычисляющем сумму, будет идти первым!

У такого единообразия трактовки функций есть рад преимуществ. Например, вам не нужно думать о приоритетах операторов, потому что перед вызовом функции всегда вычисляются её аргументы, поэтому выражение `(* (+ 5 7) 9)` вычисляется как "(5 + 7) * 9" и не нужно помнить, что в арифметике приоритет у умножения выше, чем у сложения.

Второе преимущество заключается в том, что у функций вроде `+` появляется возможность принимать больше (или меньше!) аргументов, чем бывает операндов у соответствующего оператора. Можно посчитать сумму пяти чисел одним вызовом функции сложения: `(+ 1 2 3 4 5)`. Или можно проверить, что все числа следуют друг за другом в порядке возрастания: `(< 5 7 9)`. В частности, сравнение трёх аргументов можно часто встретить в ситуации, когда нужно проверить, входит ли число в интервал: `(if (<= 0 x 10) :yes :no)` читается проще, чем `(if (and (<= 0 x) (<= x 10)) ...)`, не правда ли?

## Цитирование и децитирование

Цитирование уже упоминалось выше в разделе про символы как способ приостановить замену символа на связанное с ним значение. Но цитирование используется не только с символами, оно работает в любом месте кода и останавливает вычисление всего выражения, которое идёт следом за символом одиночной кавычки. Например, если вы введёте `'(+ 1 2)` в REPL, вы не получите тройку, вместо этого будет напечатан список из трёх элементов, первый из которых — символ "+". Заметьте, именно символ, а не функция сложения, потому что замена имени на значение — тоже часть вычисления! Именно поэтому в цитате можно указывать символы, которые ещё не определены: `'(if foo (println bar) (explode))` — вы можете ввести это выражение в REPL и не получите в ответ сообщений о том, что "foo" и "bar" не определены.

Цитирование широко используется в метапрограммировании, потому что вам постоянно приходится приостанавливать вычисление, чтобы поработать со списками как с данными, а потом уже выполнять полученный код как обычно. Выполнение кода осуществляется вызовом функции `eval`, которая вычисляет то, что запретила вычислять кавычка, и `(eval '(+ 1 2))` вычислится в число "3".

Попробуйте процитировать выражения разной вложенности и посмотрите, что выводится, если эти цитаты вычислить с помощью `eval`. Можете, скажем, поизучать выражение `'(+ (eval '(* 3 4)) 5)` — посмотрите, выполнится ли вложенный `eval` до того, как вы форсируете вычисление всей внешней цитаты?

В разделе, рассказывающем про списки, было сказано, что любой список трактуется как вызов функции, если литерал списка не процитировать. Но как быть, если список нужен как структура данных, но не хочется откладывать вычисление его элементов? Как, к примеру, получить именно список из чисел 1, 2 и 3 из литерала `'(1 (+ 1 1) 3)`, а не иметь `(+ 1 1)` вместо двойки? Если вы встретитесь с такой задачей, воспользуйтесь функцией `list`, которая создаёт список, но не вычисляет его как вызов функции, ведь она сама выступает в роли таковой: выражение `(list 1 (+ 1 1) 3)` вычислится в список `'(1 2 3)` — можете проверить в REPL.

## Определения

*Определение* (*definition*) чего либо в Clojure — это связывание некоторого значения с именем. Именем всегда выступает символ, а вот значения могут быть самых разных типов. Самый простой вид определения — это определение константы, то есть единоразовое и постоянное связывание значения и имени. Выглядят такие определения следующим образом:

``` clojure
;; RGB-цвет, представленный как вектор из трёх чисел,
;; записанных в 16-ичной сисмете исчисления
(def fuchsia [0xFF 0x00 0xFF])

(def pi
  "A number Pi" ; это строка документации (docstring)
  3.1415926)    ; а это уже значение

;; документацию по символу можно посмотреть в REPL,
;; вызвав функцию (doc SYMBOL)
(doc pi)
(doc list)
```

Определения функций выглядят так:

``` clojure
(defn foo [x y]   ; параметры функции
  (+ x (* y 10))) ; x + y * 10
```

Заметьте, что *формальные параметры* функции описываются как вектор. А само определние выглядит как список, а точнее как вызов какой-то функции. Если вы тоже так подумали, то интуиция вас не подвела: такие определения функций являются вызовами — вызовами макроса `defn`, который преобразует примерно во что-то такое:

``` clojure
(def foo (fn [x y]
           (+ x (* y 10))))
```

Как видите, это тоже определение константы, только в виде значения выступает литерал, описывающий *анонимную функцию* или, как её ещё называют, "лямбда-функцию" или просто "лямбду" (название восходит к Лямбда-исчислению, это такая математическая модель).

Довольно часто в функциональном программировании вам требуется функция, которая нужна ровно в одном месте, и анонимные функции как раз в таких случаях и используюся. В некоторых языках именнованные функции и анонимные представлены разными сущностями. А в Clojure и здесь всё привычно единообразно: именованные функции — это всего лишь константы с анонимными функциями в роли значений.

Строго говоря, макрос `defn` не только переписывает определение, но и выполняет некоторые другие преобразования. Но пока мы их рассматривать не будем. Как только нам понядобятся специфичные возможности `defn`, о них будет рассказано.
