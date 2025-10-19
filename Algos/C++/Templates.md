Шаблоны (темплейты) — это фича языка для обобщённого программирования (generic programming) — написания функций, которые обобщённо могут обрабатывать разные типы данных.

Банально, вместо этого:

```c++
char max(char a, char b)
{
	return (a >= b ? a : b);
}

unsigned char max(unsigned char a, unsigned char b)
{
	return (a >= b ? a : b);
}

short int max(short int a, short int b)
{
	return (a >= b ? a : b);
}

unsigned short int max(unsigned short int a, unsigned short int b)
{
	return (a >= b ? a : b);
}

int max(int a, int b)
{
	return (a >= b ? a : b);
}

unsigned int max(unsigned int a, unsigned int b)
{
	return (a >= b ? a : b);
}

// ... и т.д. для всех числовых типов, включая "float" и "double"...

int main()
{
	const int a = 3, b = 2, c = 1;
	const int abMaxInt = max(a, b);
	const int maxInt = max(abMax, c);
  
  // ...зато теперь можно получить максимальный "char"
  const char aChar = 'c', bChar = 'b', cChar = 'a';
	const char abMaxChar = max(aChar, bChar);
	const char maxChar = max(abMaxChar, cChar);

	return 0;
}
```

Вот это:

```c++
template<typename Type>
Type max(Type a, Type b)
{
	return (a >= b ? a : b);
}

int main()
{
  // Использование шаблона "max<Type>(Type, Type)" с подстановкой "int"
	const int a = 3, b = 2, c = 1;
	const int abMax = max<int>(a, b);
	const int max = max<int>(abMax, c);

  // Использование того же шаблона max<Type>(Type, Type) с подстановкой "char"  
	const char aChar = 3, bChar = 2, cChar = 1;
	const char abMaxChar = max<char>(aChar, bChar);
	const char maxChar = max<char>(abMaxChar, cChar);  
  
	return 0;
}
```

Перегрузки функций с повторяющийся логикой заменились на одну "функцию" с новой конструкцией - `template<typename Type>`. Слово "функция" взято тут в кавычки намеренно. Это не совсем функция. Данная запись означает для компилятора следующее: "После конструкции `template<typename Type>` описан _шаблон функции_, по которому _подстановкой_ типа вместо _шаблонного аргумента_ Type _порождаются_ конкретные функции".

Аналогичный синтаксис для классов:

```c++
template <typename T>
class DynamicArray {
private:
	int size;
	int capacity;
	T* data;
	
public:
	DynamicArray(int s, int c = 0) {
		size = s;
		(capacity == 0) ? capacity = s : capacity = c;
		data = new T[c];
	}

	T& operator[](int index) {
		if (index < 0 || index >= size) {
			throw out_of_range("Index out of bounds");
		}
		return data[index];
	}

	// Далее методы для динамического массива...
};
```

Также, если определённый тип из множества типов для шаблона нуждается в особой обработке, можно явно это указать

```c++ 
// У шаблона всегда должно быть привычное нам, обобщённое описание. Оно будет
// выбираться при подстановке в случае если ни одна специализация не подойдёт.
template<typename Type>
class SimpleArray
{
	//...
};

// Ниже описывается _специализация шаблона_. В случае, если в SimpleArray в
// качестве "Type" передаётся "bool" ("SimpleArray<bool>"), будет выбрано именно
// это описание шаблона.
template<> // [1]
class SimpleArray<bool> // [2]
{
	//...
};

// [1] – тут можно задать дополнительные шаблонные аргументы, от которых
//  зависит специализация. Этот механизм необходим для более сложных шаблонных
//  конструкций: для так называемых _частичных специализаций_ (partial
//  specialization). Мы немного коснёмся этой темы в последнем разделе.
//
// [2] – тут определяется, собственно, правило выбора данной специализации. В
//  данном случае оно очень простое: специализация выбирается если в качестве
//  значения шаблонного аргумента "Type" в "template<Type> class SimpleArray"
//  передаётся тип "bool".
//
// Специализаций по разным типам может быть сколько угодно. Например, если бы
// это имело смысл, можно было бы описать ещё одну специализацию:
//
// template<>
// class SimpleArray<int>
// {
// 	//...
// };
//
// Она выбиралась бы, если бы в качестве "Type" передавался тип "int".
```

Изначально специализации создавались как механизм для выбора оптимальной реализации шаблона для конкретного типа. Однако позже они начали исполнять одну из ключевых ролей в метапрограммировании на C++. Возможность выбирать вариации шаблона по передаваемому типу позволяет анализировать типы в программе, что открывает доступ к рефлексии времени компиляции.
## Проверка типов шаблонов

> [!WARNING] До C++ 20 не было проверки шаблонов на соответствие типам
> Поэтому функция или метод класса заявленный на `<int>` обрабатывал всё как `<int>` даже `char`

...поэтому в C++ 20 ввели концепты — `concepts`, которые проверяют совместимость типов во время компиляции

```c++
#include <cstddef>
#include <concepts>
#include <functional>
#include <string>
 
// Объявление концепта "Хэш-таблица", который удовлетворяет любой тип "T"
// такой, что для всех его значений "a" типа "T", выражение std::hash<T>{}(a)
// компилируется и результат может быть преобразован в std::size_t 
template<typename T>
concept Hashable = requires(T a)
{
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};
 
struct meow {};
 
// Введение ограничения:
template<Hashable T>
void f(T) {}
//
// Альтернтативный метод:
// template<typename T>
//     requires Hashable<T>
// void f(T) {}
//
// template<typename T>
// void f(T) requires Hashable<T> {}
//
// void f(Hashable auto /* parameter-name */) {}
 
int main()
{
    using std::operator""s;
 
    f("abc"s);    // Ок, удовлетворяет Хэш-таблицам
    // f(meow{}); // Ошибка: Не удовлетворяет хэш-таблицам
}
```

Ещё пример для (упрощённого) контейнера:

```c++
#include <iostream>
#include <concept>
#include <vector>

template<class T>
concept HasBeginEnd = 
    requires(T a) {
        a.begin();
        a.end();
    }; // Должен иметь начало и конец (т.е. они должны быть true)

template<HasBeginEnd T>
void Print(std::ostream& out, const T& v) {
    for (const auto& elem : v) {
        out << elem << std::endl;
    }
}

template<class T>
void Print(std::ostream& out, const T& v) {
    out << v;
}
```
  
_Концепт — это имя для ограничения._  
  
_Ограничение — это шаблонное булево выражение._  
  
Грубо говоря, приведённые выше условия «быть итератором» или «являться числом с плавающей точкой» — это и есть ограничения. Вся суть нововведения заключается именно в ограничениях, а концепт — лишь способ на них ссылаться.

