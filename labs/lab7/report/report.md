---
## Front matter
title: "Отчёт по лабораторной работе №7. Дискретное логарифмирование в конечном поле"
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

Ознакомиться с алгоритмом дискретного логарифмирования в конечном поле.

## Задание [@lab-task]

1. Реализовать алгоритм дискретного логарифмирования в конечном поле;
2. Вычислить логарифм с заданными числами $p,a,b$.

# Теоретическое введение [@infobez-course]

Алгоритм Полларда — это эффективный метод для нахождения дискретного логарифма, который использует идею случайных блужданий и метод "кролика и черепахи".

## Определение задачи

Дискретный логарифм — это задача нахождения целого числа x по заданным элементам g и y в конечной группе G, такой что:

$$ g^x \equiv y (mod p)$$

где g — основание, y — результат возведения в степень, а p — простое число, определяющее порядок группы.

## Основная идея алгоритма

Алгоритм Полларда основан на методе случайных блужданий и использует принцип "кролика и черепахи" (или "метод Флойда").
Он предполагает, что мы можем генерировать последовательности значений с помощью случайных блужданий и сравнивать их для нахождения совпадений.

Псевдокод, описывающий работу алгоритма:

```
input: a: a generator of G
       b: an element of G
output: An integer x such that a^x = b, or failure

Initialise i ← 0, a_0 ← 0, b_0 ← 0, x_0 ← 1 ∈ G

loop
    i ← i + 1

    x_i ← f(x_i−1),
    a_i ← g(x_i−1, a_i−1),
    b_i ← h(x_i−1, b_i−1)

    x_2i−1 ← f(x_2i−2),
    a_2i−1 ← g(x_2i−2, a_2i−2),
    b_2i−1 ← h(x_2i−2, b_2i−2)
    x_2i ← f(x_2i−1),
    a_2i ← g(x_2i−1, a_2i−1),
    b_2i ← h(x_2i−1, b_2i−1)
while x_i ≠ x_2i

r ← b_i − b_2i
if r = 0 return failure
return r−1(a_2i − a_i) mod n
```

Алгоритм Полларда является мощным инструментом для решения задачи дискретного логарифмирования в конечных группах.
Его эффективность в основном зависит от выбора начальных значений и структуры группы.
Этот алгоритм часто используется в криптографии, особенно в контексте систем на основе эллиптических кривых и других методов шифрования.

# Выполнение лабораторной работы [@lab-task]

## Реализовать алгоритм дискретного логарифмирования в конечном поле

Исходный код написан на языке `Julia` [@doc-julia]. Код функции, осуществляющей алгоритм дискретного логарифмирования в конечном поле, представлен ниже.

```julia
function new_xab(x, a, b, p, alph, bett)
    if x % 3 == 0
        return x^2 % p, a*2 % (p-1), b*2 % (p-1)
    elseif x % 3 == 1
        return x*alph % p, (a+1) % (p-1), b
    else
        return x*bett % p, a, (b+1) % (p-1)
    end
end

function searching_for_gamma(a_diff, b_diff, p)
    for i in 1:p
        if b_diff*i % p == a_diff
            return i
        end
    end
    return "Not found"
end

function metodPollarda(p, alp, bet)# , any_func::Function)
    if p % 2 == 0
        return "Incorrect input: p must be simple"
    end
    a_i = 0; b_i = 0; x_i = 1
    a_2i = 0; b_2i = 0; x_2i = 1
    i = 1
    tries = 1000
    data = zeros(Int64, (3, tries))
    data2 = zeros(Int64, (3, tries))
    while i <= tries
        x_i, a_i, b_i = new_xab(x_i, a_i, b_i, p, alp, bet)
        data[:, i] = [x_i, a_i, b_i]
        
        x_2i, a_2i, b_2i = new_xab(x_2i, a_2i, b_2i, p, alp, bet)
        x_2i, a_2i, b_2i = new_xab(x_2i, a_2i, b_2i, p, alp, bet)
        data2[:, i] = [x_2i, a_2i, b_2i]

        if x_i == x_2i
            display(data[:, 1:i])
            display(data2[:, 1:i])
            r = b_2i - b_i
            if r == 0
                return "Не найдено"
            else
                return searching_for_gamma(a_i - a_2i, r, p)
            end
        end
        i += 1
    end
    return "Делитель не найден"
end
```

### Проверка работы функции

Для проверки работы функции проверим работу функции на лёгких значениях:

```julia
p = 1019
alp = 2
bet = 5
metodPollarda(p, alp, bet)
```

Результат работы кода представлен ниже (рис. [-@fig:001]).

![Результат работы реализованной функции разложения числа на множители](image/2.png){#fig:001 width=70%}

## Вычисление логарифма с заданными числами $p,a,b$

```julia
p = 107
alp = 10
bet = 64
metodPollarda(p, alp, bet)
```

Результат работы кода представлен ниже (рис. [-@fig:002]).

![Результат работы реализованной функции разложения числа на множители](image/1.png){#fig:002 width=70%}

# Выводы

В результате работы мы ознакомились с алгоритмом дискретного логарифмирования в конечном поле и реализовали его на языке программирования `Julia`.

Также были записаны скринкасты:

На RuTube:

- [Весь плейлист](https://rutube.ru/plst/540770)
- [Запись создания шаблона отчёта и презентации для заполнения](https://rutube.ru/video/f2eff0bf79aae34ebe62602bdb92a9b8)
- [Выполнения лабораторной работы](https://rutube.ru/video/be5076520a95bc8f7a83720adba662ff)
- [Запись создания отчёта](https://rutube.ru/video/991104f7db162156d499eb7572878631)
- [Запись создания презентации](https://rutube.ru/video/28ab193ab31144b67f5984e370392282)
- [Защита лабораторной работы](https://rutube.ru/video/317a3461aefa7f67121e6b61a7c400be)

На Платформе:

- [Весь плейлист](https://plvideo.ru/playlist?list=vaNN02mO97J6)
- [Запись создания шаблона отчёта и презентации для заполнения](https://plvideo.ru/watch?v=xAma7VEEbvb-)
- [Выполнения лабораторной работы](https://plvideo.ru/watch?v=1kSw7LKQhOl-)
- [Запись создания отчёта](https://plvideo.ru/watch?v=RR6a43t3hZEV)
- [Запись создания презентации](https://plvideo.ru/watch?v=Ailpu5FuSQ2C)
- [Защита лабораторной работы](https://plvideo.ru/watch?v=0Iq7_OQGMLgB)

# Список литературы{.unnumbered}

::: {#refs}
:::
