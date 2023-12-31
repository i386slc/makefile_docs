# Цели (targets)

## Все цели

Создаете несколько целей и хотите, чтобы все они выполнялись? Сделайте цель **all**. Поскольку это первое правило в списке, оно будет выполняться по умолчанию, если **make** вызывается без указания цели.

```makefile
all: one two three

one:
	touch one
two:
	touch two
three:
	touch three

clean:
	rm -f one two three
```

## Множественные цели

При наличии нескольких целей для правила команды будут выполняться для каждой цели. **$@** — это [автоматическая переменная](avtomaticheskie-peremennye-i-podstanovochnye-znaki-wildcards.md#avtomaticheskie-peremennye), содержащая имя цели.

```makefile
all: f1.o f2.o

f1.o f2.o:
	echo $@

# Эквивалентно:
# f1.o:
#	 echo f1.o
# f2.o:
#	 echo f2.o
```
