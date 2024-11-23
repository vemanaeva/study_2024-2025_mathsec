---
## Front matter
title: "Отчёт по лабораторной работе №6. Разложение числа на множители"
subtitle: "Дисциплина: Математические основы защиты информации и информационной безопасности"
author: "Манаева Варвара Евгеньевна"

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
lot: false # List of tables
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
mainfont: IBM Plex Serif
romanfont: IBM Plex Serif
sansfont: IBM Plex Sans
monofont: IBM Plex Mono
mathfont: STIX Two Math
mainfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
romanfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
sansfontoptions: Ligatures=Common,Ligatures=TeX,Scale=MatchLowercase,Scale=0.94
monofontoptions: Scale=MatchLowercase,Scale=0.94,FakeStretch=0.9
mathfontoptions:
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

# Общая информация о задании лабораторной работы

## Цель работы

Ознакомиться с алгоритмами разложения числа на множители.

## Задание [@lab-task]

1. Задание.

# Теоретическое введение [@infobez-course]

## Разложение на множители

pho-метод Полланда (или $\rho-1$ метод Полларда) является одним из алгоритмов для факторизации целых чисел, который особенно эффективен для нахождения малых простых делителей. 
Он основан на свойствах чисел и использует последовательности, чтобы вычислить делители.

### Основные этапы метода

1. Подготовка: 
   - **Выбор числа n:** Начинаем с целого числа n, которое необходимо факторизовать; 
   - **Выбор параметров:** Выбираем небольшое целое число a и границу B, которая будет использоваться для ограничения множителей.
2. Генерация последовательности: Создаем последовательность чисел по формуле: $x_{k+1} = (x_k^2 + a)$.
3. Вычисление НОД: На каждом шаге вычисляем наибольший общий делитель (НОД) между n и разностью двух членов последовательности.
4. Проверка результата: Если найденный НОД d больше 1 и меньше n, то это делитель числа n. Если $d = n$, то алгоритм не дал результата, и его можно повторить с другими параметрами. Если $d=1$, то повторяем действия со второго шага.
5. Завершение: Процесс продолжается до тех пор, пока не будет найден делитель или не исчерпаются все возможные варианты.

### Применение метода

Метод Полланда эффективен для нахождения малых простых делителей, особенно когда число имеет структуру, позволяющую выделить
такие делители. Он также может быть использован в сочетании с другими методами факторизации для повышения общей эффективности.

# Выполнение лабораторной работы [@lab-task]

Исходный код написан на языке `Julia` [@doc-julia]. Код функции, осуществляющей разложение числа на множители, представлен ниже.

```julia
function metodPollarda(n, c, any_func::Function)
    if n % 2 == 0
        return 2, round(Int, n/2)
    end
    a = c; b = c
    i = 0
    while i < 100
        a = any_func(a)
        b = any_func(any_func(b))
        d = evklidBin(a-b, n)
        # println(a, "\t", b, "\t", d)
        if d > 1
            return d, round(Int, n/d)
        end
        i += 1
    end
    return "Делитель не найден"
end
```

Разберём подробно работу функции.

На вход функция принимает 3 параметра: 

- `n` -- число, которое необходимо факторизовать;
- `c` -- число, которое используется в качестве начала отсчёта;
- `any_func::Function` -- функция, по которой рассчитывается каждая следующая итерация.

Функцию саму можно поделить на несколько смысловых частей:

1. Предобработка;
2. Входящие параметры для цикла;
3. Цикл работы функции;
4. Вывод при неудачном наборе входящих данных.

### 1. Предобработка

Если число, которое необходимо факторизовать, делится на 2, то оно не подходит под действие алгоритма (на вход даётся только нечётное число), в связи с чем можно сразу вывести делители
этого числа.

```julia
if n % 2 == 0
    return 2, round(Int, n/2)
end
```

### 2. Входящие параметры для цикла

Первым шагом алгоритма является подготовка двух промежуточных значений (`a` и `b`), которые будут представлять $x_i$ и $x_{2i}$ в рамках работы алгоритма.
Также задаётся счётчик для ограничения числа итераций работы функции.

```julia
a = c; b = c
i = 0
```

### 3. Цикл работы функции

Основный цикл работы функции, включающий в себя шаги 2-4 работы алгоритма.

```julia
while i < 100
    a = any_func(a)
    b = any_func(any_func(b))
    d = evklidBin(a-b, n)
    # println(a, "\t", b, "\t", d)
    if d > 1
        return d, round(Int, n/d)
    end
    i += 1
end
```

### 4. Вывод при неудачном наборе входящих данных

Возвращение значения "Делитель не найден" при завершении работы цикла в связи с превышением числа итераций.

```julia
return "Делитель не найден"
```

### Проверка работы функции

```julia
n = 1359331
c = 1
metodPollarda(n, c, x -> (x^2 + 5) % n)
```

Результат работы кода представлен ниже (рис. [-@fig:001]).

![Результат работы реализованной функции разложения числа на множители](image/1.png){#fig:001 width=70%}

## Разложение крупного числа на множители

```julia
n = 135956347
c = 1
metodPollarda(n, c, x -> (x^2 + 13) % n)
```

Результат работы кода представлен ниже (рис. [-@fig:002]).

![Результат работы реализованной функции разложения числа на множители](image/2.png){#fig:002 width=70%}

# Выводы

В результате работы мы ознакомились с алгоритмом разложения чисел на множители и реализовали его на языке программирования `Julia`.

Также были записаны скринкасты:

На RuTube:

- [Весь плейлист](https://rutube.ru/plst/540770)
- [Запись создания шаблона отчёта и презентации для заполнения](https://rutube.ru/video/f2eff0bf79aae34ebe62602bdb92a9b8)
- [Выполнения лабораторной работы](https://rutube.ru/video/9d1e02d6a98d3dfce4f743fd2d89843f)
- [Запись создания отчёта](https://rutube.ru/video/0fee934896169f91f5c39af6f1497750)
- [Запись создания презентации](https://rutube.ru/video/2087b47d8b02e223846b98d17832fa54)
- [Защита лабораторной работы](https://rutube.ru/video/a07854950c3721ba9effd97c191808a7)

На Платформе:

- [Весь плейлист](https://plvideo.ru/playlist?list=vaNN02mO97J6)
- [Запись создания шаблона отчёта и презентации для заполнения](https://plvideo.ru/watch?v=xAma7VEEbvb-)
- [Выполнения лабораторной работы](https://plvideo.ru/watch?v=RtQyddHA0R-U)
- [Запись создания отчёта](https://plvideo.ru/watch?v=4xzJteHE0wf-)
- [Запись создания презентации](https://plvideo.ru/watch?v=wEL-OlaFiLq6)
- [Защита лабораторной работы](https://plvideo.ru/watch?v=zWEXQ6OOyyLE)

# Список литературы{.unnumbered}

::: {#refs}
:::
