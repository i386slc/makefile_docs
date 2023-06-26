# Функции

## Первые функции

Функции в основном только для обработки текста. Вызывайте функции с `$(fn, arguments)` или `${fn, arguments}`. Make имеет приличное количество [встроенных функций](https://www.gnu.org/software/make/manual/html\_node/Functions.html).

```makefile
bar := ${subst not, totally, "I am not superman"}
all: 
	@echo $(bar)
```

Если вы хотите заменить пробелы или запятые, используйте переменные

```makefile
comma := ,
empty:=
space := $(empty) $(empty)
foo := a b c
bar := $(subst $(space),$(comma),$(foo))

all: 
	@echo $(bar)
```

НЕ включайте пробелы в аргументы после первого. Это будет рассматриваться как часть строки.

```makefile
comma := ,
empty:=
space := $(empty) $(empty)
foo := a b c
bar := $(subst $(space), $(comma) , $(foo))

all: 
	# Выведет ", a , b , c". Обратите внимание на введенные пробелы
	@echo $(bar)
```

## Замена строки

`$(patsubst pattern,replacement,text)` делает следующее:

> «Находит в **text** слова, разделенные пробелами, которые соответствуют шаблону **pattern**, и заменяет их **replacement**. Здесь pattern может содержать `«%»`, который действует как подстановочный знак, соответствующий любому количеству любых символов в слове. Если replacement также содержит `«%»`, `'%'` заменяется текстом, который соответствует `'%'` в pattern. Только первый `'%'` в pattern и replacement  обрабатывается таким образом; все последующие `'%'` не меняются." ([документация GNU](https://www.gnu.org/software/make/manual/html\_node/Text-Functions.html#Text-Functions))

Ссылка на замену `$(text:pattern=replacement)` является сокращением для этого.

Есть еще одно сокращение, заменяющее только суффиксы: `$(text:suffix=replacement)`. Здесь не используется подстановочный знак `%`.

{% hint style="info" %}
Не добавляйте дополнительные пробелы для этого сокращения. Он будет рассматриваться как поисковый или замещающий термин.
{% endhint %}

```makefile
foo := a.o b.o l.a c.o
one := $(patsubst %.o,%.c,$(foo))
# Это сокращение от вышеизложенного
two := $(foo:%.o=%.c)
# Это сокращение, состоящее только из суффиксов, также эквивалентно приведенному выше
three := $(foo:.o=.c)

all:
	echo $(one)
	echo $(two)
	echo $(three)
```

## Функция foreach

Функция **foreach** выглядит так: `$(foreach var,list,text)`.

Он преобразует один список слов (разделенных пробелами) в другой. **var** устанавливается для каждого слова в списке, и **text** расширяется для каждого слова.

Это добавляет восклицательный знак после каждого слова:

```makefile
foo := who are you
# Для каждого "word" в foo, выведет это же слово с восклицательным знаком в конце
bar := $(foreach wrd,$(foo),$(wrd)!)

all:
	# Выведет "who! are! you!"
	@echo $(bar)
```

## Функция if

**if** проверяет, не пуст ли первый аргумент. Если это так, запускается второй аргумент, в противном случае запускается третий.

```makefile
foo := $(if this-is-not-empty,then!,else!)
empty :=
bar := $(if $(empty),then!,else!)

all:
	@echo $(foo)
	@echo $(bar)
```

## Функция call

Make поддерживает создание основных функций. Ваша функция `"define"`, просто создает переменную, но используя параметры `$(0)`, `$(1)` и т. д. Затем вы вызываете функцию с помощью специальной встроенной функции **call**.

Синтаксис: `$(call variable,param,param)`.

**$(0)** — переменная, а **$(1)**, **$(2)** и т. д. — параметры.

```makefile
sweet_new_fn = Variable Name: $(0) First: $(1) Second: $(2) Empty Variable: $(3)

all:
	# Выводит "Variable Name: sweet_new_fn First: go Second: tigers Empty Variable:"
	@echo $(call sweet_new_fn, go, tigers)
```

## Функция оболочки shell

**shell** — вызывает оболочку, но заменяет символы новой строки пробелами!

```makefile
all: 
	@echo $(shell ls -la) # Очень некрасиво, потому что новые строки исчезли!
```
