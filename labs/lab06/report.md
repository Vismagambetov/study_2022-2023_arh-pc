---
## Front matter
title: "Отчёта по лабораторной работе 6"
subtitle: "Освоение арифметических инструкций языка ассемблера NASM."
author: "Исмагамбетов Владимир НБИбд-04-22"

## Generic otions
lang: ru-RU
toc-title: "Содержание"

## Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

## Pdf output format
toc: true # Table of contents
toc-depth: 2
lof: true # List of figures
lot: true # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a4
documentclass: scrreprt
## I18n polyglossia
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
mainfont: PT Serif
romanfont: PT Serif
sansfont: PT Sans
monofont: PT Mono
mainfontoptions: Ligatures=TeX
romanfontoptions: Ligatures=TeX
sansfontoptions: Ligatures=TeX,Scale=MatchLowercase
monofontoptions: Scale=MatchLowercase,Scale=0.9
## Biblatex
biblatex: true
biblio-style: "gost-numeric"
biblatexoptions:
  - parentracker=true
  - backend=biber
  - hyperref=auto
  - language=auto
  - autolang=other*
  - citestyle=gost-numeric
## Pandoc-crossref LaTeX customization
figureTitle: "Рис."
tableTitle: "Таблица"
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lotTitle: "Список таблиц"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Цель работы

Целью работы является освоение арифметических инструкций языка ассемблера NASM.


# Задание

1. Изучите примеры программ.

2. Напишите программу вычисления выражения в соответсвии с вариантом.

3. Загрузите файлы на GitHub.

# Теоретическое введение

В основном наборе инструкций входят разные вариации четырех арифметических действий: 
сложение, вычитание, умножение, деление. Важно помнить, что в результате арифметических 
действий меняются некоторые биты регистра флагов, что позволяет выполнять команду условного 
перехода, т.е. разветвлять программу на основе результат операции. Замечу, что для команд с
ложения и вычитания справедливыми являются отмеченное выше для операндов команды mov . 
К командам сложения можно отнести: add – обычное сложение, 
adc – сложение с добавлением результату флага переноса в качестве единицы 
(если флаг равен нулю, то команда эквивалентна команде add ), 
xadd – сложение, с предварительным обменом данных между операндами, 
inc – прибавление единицы к содержимому операнда. 
Несколько примеров: add %rbx , dt (или addq, dt, где четко указано, что складываются 
64-битовые величины) – к содержимому области памяти dt добавляется содержимое регистра 
rbx и результат помещается в dt ; adc %rdx , %rdx – удвоение содержимого регистра rdx плюс 
добавление значения флага переноса; incl ll – увеличение на единицу содержимого памяти по 
адресу ll. При этом явно указывается, что операнд имеет размер 32 бита (d - dword).

К командам вычитания можно отнести следующие инструкции процессора x86-64: 
sub – обычное вычитание, 
sbb - вычитание из результата флага переноса в качестве единицы 
(если флаг равен нулю, то команда эквивалентна sub ), 
dec – вычитание единицы из результата, 
neg – вычитание значения операнда из 0 . 
Несколько примеров: sub %rax , ll - из содержимого ll вычитается содержимое регистра 
rax (или явно subq %rax , ll, где указывается, что операнды имеют 64-размер), 
и результат помещается в ll; subw go, %ax – вычитание из содержимого ax числа по адресу go, 
результат помещается в ax ; sbb %rdx , %rax – вычитание с дополнительным вычитанием флага 
переноса (из числа в rax вычитается число в rdx и результат в rax); 
decb l – вычитание единицы из байта, расположенного по адресу l . Следует отметить 
еще специальную команду cmp , которая во всем похожа на команду sub, 
кроме одного – результат вычитания никуда не помещается. Инструкция используется специально, 
для сравнения операндов.

Две основные команды умножения: mul – умножение беззнаковых чисел, 
imul – умножение знаковых чисел. Команда содержит один операнд – регистр или адрес памяти. 
В зависимости от размера операнда данные помещаются: в ax , dx : ax , edx : eax , rdx : rax . 
Например: mull ll – содержимое памяти с адресом ll будет умножено на содержимое eax 
(не забываем о суффиксе l), а результат отправлен в пару регистров edx : eax; 
mul %dl – умножить содержимое регистра dl на содержимое регистра al , 
а результат положить в ax ; mul %r8 – умножить содержимое регистра r8 на содержимое регистра 
rax , а результат положить в пару регистров rdx : rax.

Для деления (целого) также предусмотрены две команды: div – беззнаковое деление, 
idiv – знаковое деление. Инструкция также имеет один операнд - делитель. 
В зависимости от его размера результат помещается: al – результат деления, 
ah – остаток от деления; ax – результат деления, dx – остаток от деления; 
eax – результат деления, edx – остаток от деления; rax – результат деления, 
rdx – остаток от деления. Приведем примеры: divl dv – содержимое edx : eax делится на делитель, 
находящийся в памяти по адресу dv и результат деления помещается в eax , остаток в edx ; 
div %rsi – содержимое rdx : rax делится на содержимое rsi , результат помещается в rax , 
остаток в rdx .

# Выполнение лабораторной работы

1. Создайте каталог для программам лабораторной работы № 6, перейдите в него и создайте файл lab7-1.asm: 

2. Рассмотрим примеры программ вывода символьных и численных значений. 
Программы будут выводить значения, записанные в регистр eax. (рис. [-@fig:001], [-@fig:002])

![Пример программы](image/01.png){ #fig:001 width=70%, height=70% }

![Работа программы](image/02.png){ #fig:002 width=70%, height=70% }

3. Далее изменим текст программы и вместо символов, запишем в регистры числа. 
Исправьте текст программы (Листинг 1) следующим образом: (рис. [-@fig:003], [-@fig:004])

![Пример программы](image/03.png){ #fig:003 width=70%, height=70% }

![Работа программы](image/04.png){ #fig:004 width=70%, height=70% }

Никакой символ не виден, но он есть. Это возврат каретки LF.

4. Как отмечалось выше,для работы с числами в файле in_out.asm реализованы 
подпрограммы для преобразования ASCII символов в числа и обратно. 
Преобразуем текст программы из Листинга 7.1 с использованием этих функций. (рис. [-@fig:005], [-@fig:006])

![Пример программы](image/05.png){ #fig:005 width=70%, height=70% }

![Работа программы](image/06.png){ #fig:006 width=70%, height=70% }

В результате работы программы мы получим число 106. В данном случае, как и в первом, 
команда add складывает коды символов ‘6’ и ‘4’ (54+52=106). 
Однако, в отличии от программы из листинга 7.1, функция iprintLF позволяет вывести число, 
а не символ, кодом которого является это число.

5. Аналогично предыдущему примеру изменим символы на числа. (рис. [-@fig:007], [-@fig:008])

Создайте исполняемый файл и запустите его. 
Какой результат будет получен при исполнении программы? – получили число 10

![Пример программы](image/07.png){ #fig:007 width=70%, height=70% }

![Работа программы](image/08.png){ #fig:008 width=70%, height=70% }

Замените функцию iprintLF на iprint. Создайте исполняемый файл и запустите его. 
Чем отличается вывод функций iprintLF и iprint? - Вывод отличается что нет переноса строки. (рис. [-@fig:009])

![Работа программы](image/09.png){ #fig:009 width=70%, height=70% }

6. В качестве примера выполнения арифметических операций в NASM приведем 
программу вычисления арифметического выражения 
$$f(x) = (5 * 2 + 3)/3$$. (рис. [-@fig:010], рис. [-@fig:011])

![Пример программы](image/10.png){ #fig:010 width=70%, height=70% }

![Работа программы](image/11.png){ #fig:011 width=70%, height=70% }

Измените текст программы для вычисления выражения 
$$f(x) = (4 * 6 + 2)/5$$. 
Создайте исполняемый файл и проверьте его работу. (рис. [-@fig:012], рис. [-@fig:013])

![Пример программы](image/12.png){ #fig:012 width=70%, height=70% }

![Работа программы](image/13.png){ #fig:013 width=70%, height=70% }

7. В качестве другого примера рассмотрим программу вычисления варианта задания по 
номеру студенческого билета, работающую по следующему алгоритму: (рис. [-@fig:014], рис. [-@fig:015])

![Пример программы](image/14.png){ #fig:014 width=70%, height=70% }

![Работа программы](image/15.png){ #fig:015 width=70%, height=70% }

* Какие строки листинга 7.4 отвечают за вывод на экран сообщения ‘Ваш вариант:’? – mov eax,rem – перекладывает в регистр значение переменной с фразой ‘Ваш вариант:’ call sprint – вызов подпрограммы вывода строки

* Для чего используется следующие инструкции? 
```nasm 
mov ecx, x 
mov edx, 80 
call sread```
  
Считывает значение студбилета в переменную Х из консоли

* Для чего используется инструкция “call atoi”?  - эта подпрограмма переводит введенные символы в числовой формат

* Какие строки листинга 7.4 отвечают за вычисления варианта? 
```xor edx,edx
mov ebx,20
div ebx```

* В какой регистр записывается остаток от деления при выполнении инструкции “div ebx”? 
```1 байт AH 
2 байта DX 
4 байта EDX – наш случай```

* Для чего используется инструкция “inc edx”?  по формуле вычисления варианта нужно прибавить единицу

* Какие строки листинга 7.4 отвечают за вывод на экран результата вычислений
mov eax,edx – результат перекладывается в регистр eax
call iprintLF – вызов подпрограммы вывода

8. Написать программу вычисления выражения y = f(x). Программа должна выводить выражение 
для вычисления, выводить запрос на ввод значения x, 
вычислять заданное выражение в зависимости от введенного x, выводить результат вычислений. 
Вид функции f(x) выбрать из таблицы 6.3 вариантов заданий в соответствии с номером 
полученным при выполнении лабораторной работы. 
Создайте исполняемый файл и проверьте его работу для значений x1 и x2 из 6.3. (рис. [-@fig:016], рис. [-@fig:017])

Получили вариант 4 - $$(4/3)(x - 8) + 5$$  для х=4 и 10

![Пример программы](image/16.png){ #fig:016 width=70%, height=70% }

![Работа программы](image/17.png){ #fig:017 width=70%, height=70% }

# Выводы

Изучили работу с арифметическими операциями

# Список литературы{.unnumbered}

1. [Расширенный ассемблер: NASM](https://www.opennet.ru/docs/RUS/nasm/)
2. [MASM, TASM, FASM, NASM под Windows и Linux](https://habr.com/ru/post/326078/)









