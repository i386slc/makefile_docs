# Условная часть Makefiles

## Условие if/else

```makefile
foo = ok

all:
ifeq ($(foo), ok)
	echo "foo equals ok"
else
	echo "nope"
endif
```

## Проверить, пуста ли переменная

```makefile
nullstring =
foo = $(nullstring) # конец линии; здесь есть место

all:
ifeq ($(strip $(foo)),)
	echo "foo is empty after being stripped"
endif
ifeq ($(nullstring),)
	echo "nullstring doesn't even have spaces"
endif
```

## Проверить, определена ли переменная

**ifdef** не расширяет ссылки на переменные; он просто видит, определено ли что-то вообще

```makefile
bar =
foo = $(bar)

all:
ifdef foo
	echo "foo is defined"
endif
ifndef bar
	echo "but bar is not"
endif
```

## $(MAKEFLAGS) <a href="#makeflags" id="makeflags"></a>

В этом примере показано, как тестировать флаги make с помощью **findstring** и **MAKEFLAGS**. Запустите этот пример с `make -i`, чтобы увидеть, как он распечатывает оператор **echo**.

```makefile
all:
# Ищет флаг "-i". MAKEFLAGS — это просто список отдельных символов,
# по одному на каждый флаг. Так что ищите «i» в этом случае.
ifneq (,$(findstring i, $(MAKEFLAGS)))
	echo "i was passed to MAKEFLAGS"
endif
```
