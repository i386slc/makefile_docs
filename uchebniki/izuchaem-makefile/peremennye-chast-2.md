# Переменные (часть 2)

## Особенности и модификации

Существует два вида переменных:

* рекурсивные (используйте `=`) - ищет переменные **только при использовании команды, а не при ее определении**.
* простое определение (используйте `:=`) - как обычное императивное программирование - расширяются только те, которые определены до сих пор

```makefile
# Рекурсивная переменная. Это напечатает "later" ниже
one = one ${later_variable}
# Просто расширенная переменная. Это не будет печатать "later" ниже
two := two ${later_variable}

later_variable = later

all: 
	echo $(one)
	echo $(two)
```

Простое расширение (используя `:=`) позволяет добавлять к переменной. Рекурсивные определения дадут ошибку бесконечного цикла.

```makefile
one = hello
# one определяется как просто расширенная переменная (:=)
# и, таким образом, может обрабатывать добавление
one := ${one} there

all: 
	echo $(one)
```

`?=` устанавливает переменные только в том случае, если они еще не были установлены

```makefile
one = hello
one ?= will not be set
two ?= will be set

all: 
	echo $(one)
	echo $(two)
```

Пробелы в конце строки не удаляются, а пробелы в начале удаляются. Чтобы создать переменную с одним пробелом, используйте `$(nullstring)`

```makefile
with_spaces = hello   # with_spaces имеет много пробелов после "hello"
after = $(with_spaces)there

nullstring =
space = $(nullstring) # Создает переменную с одним пробелом.

all: 
	echo "$(after)"
	echo start"$(space)"end
```

Неопределенная переменная на самом деле является пустой строкой!

```makefile
all: 
	# Неопределенные переменные — это просто пустые строки!
	echo $(nowhere)
```

Используйте `+=` для добавления

```makefile
foo := start
foo += more

all: 
	echo $(foo)
```

<mark style="color:purple;">Подстановка строк</mark> также является очень распространенным и полезным способом изменения переменных. Также ознакомьтесь с [текстовыми функциями](https://www.gnu.org/software/make/manual/html\_node/Text-Functions.html#Text-Functions) и [функциями имен файлов](https://www.gnu.org/software/make/manual/html\_node/File-Name-Functions.html#File-Name-Functions).

## Аргументы командной строки и переопределение

Вы можете переопределить переменные, поступающие из командной строки, с помощью **override**. Здесь мы запустили команду `make option_one=hi`.

```makefile
# Переопределяет аргументы командной строки
override option_one = did_override
# Не переопределяет аргументы командной строки
option_two = not_override
all: 
	echo $(option_one)
	echo $(option_two)
```

## Список команд и определение

[Директива define](https://www.gnu.org/software/make/manual/html\_node/Multi\_002dLine.html) не является функцией, хотя может так выглядеть. Я видел, как она используется так редко, что не буду вдаваться в подробности, но в основном она используется для определения [готовых рецептов](https://www.gnu.org/software/make/manual/html\_node/Canned-Recipes.html#Canned-Recipes), а также хорошо сочетается с [функцией eval](https://www.gnu.org/software/make/manual/html\_node/Eval-Function.html#Eval-Function).

`define/endef` просто создает переменную, для которой задан список команд. Обратите внимание, что это немного отличается от точки с запятой между командами, потому что каждая из них запускается в отдельной оболочке, как и ожидалось.

```makefile
one = export blah="I was set!"; echo $$blah

define two
export blah="I was set!"
echo $$blah
endef

all: 
	@echo "This prints 'I was set'"
	@$(one)
	@echo "This does not print 'I was set' because each command runs in a separate shell"
	@$(two)
```

## Переменные, зависящие от цели

Переменные могут быть установлены для конкретных целей

```makefile
all: one = cool

all: 
	echo one is defined: $(one)

other:
	echo one is nothing: $(one)
```

## Переменные, специфичные для шаблона

Вы можете установить переменные для конкретных целевых шаблонов **patterns**

```makefile
%.c: one = cool

blah.c: 
	echo one is defined: $(one)

other:
	echo one is nothing: $(one)
```
