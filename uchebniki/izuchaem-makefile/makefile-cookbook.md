# Makefile Cookbook

Давайте рассмотрим действительно сочный пример Make, который хорошо работает для проектов среднего размера.

Прелесть этого make-файла в том, что он автоматически определяет для вас зависимости. Все, что вам нужно сделать, это поместить файлы C/C++ в папку `src/`.

```makefile
# Благодарность Job Vranish
# (https://spin.atomicobject.com/2016/08/26/makefile-c-projects/)
TARGET_EXEC := final_program

BUILD_DIR := ./build
SRC_DIRS := ./src

# Ищет все файлы C и C++, которые мы хотим скомпилировать.
# Обратите внимание на одинарные кавычки вокруг выражений *.
# В противном случае оболочка неправильно расширит их,
# но мы хотим отправить * непосредственно в команду find.
SRCS := $(shell find $(SRC_DIRS) -name '*.cpp' -or -name '*.c' -or -name '*.s')

# Добавляет BUILD_DIR и добавляет .o к каждому файлу src.
# Например, ./your_dir/hello.cpp превращается в ./build/./your_dir/hello.cpp.o
OBJS := $(SRCS:%=$(BUILD_DIR)/%.o)

# Замена строки (версия суффикса без %).
# Например, ./build/hello.cpp.o превращается в ./build/hello.cpp.d
DEPS := $(OBJS:.o=.d)

# Каждая папка в ./src должна быть передана в GCC, чтобы он мог найти файлы заголовков
INC_DIRS := $(shell find $(SRC_DIRS) -type d)
# Добавьте префикс к INC_DIRS. Таким образом, модуль A станет -ImoduleA.
# GCC понимает это -I флаг
INC_FLAGS := $(addprefix -I,$(INC_DIRS))

# Флаги -MMD и -MP вместе генерируют Makefile для нас!
# Эти файлы будут иметь .d вместо .o в качестве вывода.
CPPFLAGS := $(INC_FLAGS) -MMD -MP

# Завершающий этап сборки.
$(BUILD_DIR)/$(TARGET_EXEC): $(OBJS)
	$(CXX) $(OBJS) -o $@ $(LDFLAGS)

# Шаг сборки для исходного кода C
$(BUILD_DIR)/%.c.o: %.c
	mkdir -p $(dir $@)
	$(CC) $(CPPFLAGS) $(CFLAGS) -c $< -o $@

# Шаг сборки исходного кода C++
$(BUILD_DIR)/%.cpp.o: %.cpp
	mkdir -p $(dir $@)
	$(CXX) $(CPPFLAGS) $(CXXFLAGS) -c $< -o $@


.PHONY: clean
clean:
	rm -r $(BUILD_DIR)

# Включите make-файлы .d. Знак - в начале подавляет ошибки отсутствия файлов Makefile.
# Изначально все файлы .d будут отсутствовать, и мы не хотим,
# чтобы эти ошибки появлялись.
-include $(DEPS)
```
