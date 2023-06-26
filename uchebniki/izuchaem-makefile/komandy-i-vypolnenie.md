# Команды и выполнение

Команды echo/silencing

Добавьте **@** перед командой, чтобы она не печаталась. Вы также можете запустить make с параметром `-s`, чтобы добавить **@** перед каждой строкой.

```makefile
all: 
	@echo "This make line will not be printed"
	echo "But this will"
```

## Выполнение команды

Каждая команда запускается в новой оболочке (или, по крайней мере, эффект как таковой)

```makefile
all: 
	cd ..
	# Приведенный выше cd не влияет на эту строку, потому что каждая команда
	# фактически запускается в новой оболочке.
	echo `pwd`

	# Эта команда cd влияет на следующую, потому что они находятся в одной строке
	cd ..;echo `pwd`

	# То же, что и выше
	cd ..; \
	echo `pwd`
```

## Оболочка shell по умолчанию

Оболочкой по умолчанию является `/bin/sh`. Вы можете изменить это, изменив переменную **SHELL**:

```makefile
SHELL=/bin/bash

cool:
	echo "Hello from bash"
```

## Двойной знак доллара

Если вы хотите, чтобы строка имела знак доллара, вы можете использовать **\$$**. Вот как использовать переменную оболочки в **bash** или **sh**.

Обратите внимание на различия между переменными **Makefile** и переменными оболочки в следующем примере.

```makefile
make_var = I am a make variable
all:
	# То же, что и запуск "sh_var='I am a shell variable';
	# echo $sh_var" в оболочке
	sh_var='I am a shell variable'; echo $$sh_var

	# То же, что и запуск "echo I am a make variable" в оболочке
	echo $(make_var)
```

## Обработка ошибок с помощью -k, -i и -

Добавьте `-k` при запуске **make**, чтобы продолжить работу даже при возникновении ошибок. Полезно, если вы хотите увидеть все ошибки Make сразу.

Добавьте `-` перед командой, чтобы подавить ошибку.

Добавьте `-i`, чтобы это происходило для каждой команды.

```makefile
one:
	# Эта ошибка будет напечатана, но проигнорирована, и make продолжит работу.
	-false
	touch one
```

## Прерывание или уничтожение make

Только примечание: если вы нажмете `ctrl+c`, make удалит только что созданные новые цели.

## Рекурсивное использование make

Чтобы рекурсивно вызвать make-файл, используйте специальный `$(MAKE)` вместо **make**, потому что он передаст вам флаги make и не будет зависеть от них.

```makefile
new_contents = "hello:\n\ttouch inside_file"
all:
	mkdir -p subdir
	printf $(new_contents) | sed -e 's/^ //' > subdir/makefile
	cd subdir && $(MAKE)

clean:
	rm -rf subdir
```

## Экспорт, среды и рекурсивный make

Когда Make запускается, он автоматически создает переменные Make из всех переменных среды, установленных при его выполнении.

```makefile
# Запустите это с помощью
# "export shell_env_var='Я переменная среды'; make"
all:
	# Распечатайте переменную оболочки
	echo $$shell_env_var

	# Распечатайте переменную Make
	echo $(shell_env_var)
```

Директива **export** принимает переменную и устанавливает ее в качестве среды для всех команд оболочки во всех рецептах:

```makefile
shell_env_var=Shell env var, created inside of Make
export shell_env_var
all:
	echo $(shell_env_var)
	echo $$shell_env_var
```

Таким образом, когда вы запускаете команду **make** внутри make, вы можете использовать директиву **export**, чтобы сделать ее доступной для вложенных команд make. В этом примере **cooly** экспортируется таким образом, что make-файл в подкаталоге может его использовать.

```makefile
new_contents = "hello:\n\techo \$$(cooly)"

all:
	mkdir -p subdir
	printf $(new_contents) | sed -e 's/^ //' > subdir/makefile
	@echo "---MAKEFILE CONTENTS---"
	@cd subdir && cat makefile
	@echo "---END MAKEFILE CONTENTS---"
	cd subdir && $(MAKE)

# Обратите внимание на переменные и export. Они устанавливаются/влияют глобально.
cooly = "The subdirectory can see me!"
export cooly
# Это аннулирует строку выше: unexport cooly

clean:
	rm -rf subdir
```

Вам нужно экспортировать переменные, чтобы они также запускались в оболочке.

```makefile
one=this will only work locally
export two=we can run subcommands with this

all: 
	@echo $(one)
	@echo $$one
	@echo $(two)
	@echo $$two
```

`.EXPORT_ALL_VARIABLES` экспортирует для вас все переменные.

```makefile
.EXPORT_ALL_VARIABLES:
new_contents = "hello:\n\techo \$$(cooly)"

cooly = "The subdirectory can see me!"
# Это аннулирует строку выше: unexport cooly

all:
	mkdir -p subdir
	printf $(new_contents) | sed -e 's/^ //' > subdir/makefile
	@echo "---MAKEFILE CONTENTS---"
	@cd subdir && cat makefile
	@echo "---END MAKEFILE CONTENTS---"
	cd subdir && $(MAKE)

clean:
	rm -rf subdir
```

## Аргументы для make

Есть хороший [список опций](http://www.gnu.org/software/make/manual/make.html#Options-Summary), которые можно запустить из make. Проверьте `--dry-run`, `--touch`, `--old-file`.

У вас может быть несколько целей, например `make clean run test` запускает цель **clean**, затем **run**, а затем **test**.
