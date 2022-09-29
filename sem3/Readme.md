## Массивы, динамическая память

### Санитайзеры

При вызове некоторых компиляторов можно передавать параметры, позволяющие управлять выполнением программы. 
Искать проблемы в коде помогают санитайзеры. 

[Основные санитайзеры](https://github.com/google/sanitizers/wiki/MemorySanitizer) gcc и clang:
 - AdderssSanitizer (ASan) - определяет использование уже освобожденной памяти, переполнение стека и кучи, использование переменных, определенных на стеке уже завершённой функции или в другой области видимости, утечки памяти
 - UndefinedBehaviorSanitizer (UBSan) - определнияет выход за границы массива, переполнение signed int, разыменование nullptr
 - MemorySanitizer (MSan) - определяет использование неинициализированных переменных

[GCC command line options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)

### Флаги компилятора

[Все флаги gcc](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html). [Cтатье на Хабре на русском](https://habr.com/en/post/490850/).

Самые основные:

- -Wall отображает многие предупреждения(warnings)
- -Wextra содержит расширенный список предупреждений, которые не включаются с помощью -Wall
- -Werror все предупреждения становятся ошибками компиляции
- -Wpedantic проверяет код на соответствие стандарту C++. В частоности, не позволяет создавать VLA (C-style массивы переменной длины), так как они не являются частью стандарта C++ (в отличие от C)

Что есть ещё:
- -Wreturn-type - реальный возвращемый из функции тип не соответсвует возвращаемому типу в сигнатуре функции
- -Wuninitialized - использование непроинициализированной переменной
- -Wunused-variable - переменная не используется
- --Wconversion - неявное преобразование, сужающее тип
- -Wimplicit-fallthrough - если вы забыли написать break; в swith (или если компилятор так считает) и др.

Дополнительно: [для чего нужен перенос строки в конце файла](https://semakin.dev/2020/05/no_newline_at_end_of_file/).

Рассмотрим такой очень плохой код
```C++
// main.cpp

#include <iostream>

int main() {
  int x;
  unsigned y = -1;
  int z = 9.3;

  switch(z) {
    case 8:
      std::cout << 8;
      break;
    case 9:
      std::cout << 9;
    case 10:
      std::cout << 10;
    default:
      std::cout << "kek";
  }
}
```
Какие ошибки в нём есть? Вот что может сказать нам компилятор, если мы скомпилируем программу следующим образом
c++ -Werror -Wextra -Wpedantic -Wall  -Wimplicit-fallthrough -Wconversion main.cpp:
```C++
main.cpp:5:16: error: implicit conversion changes signedness: 'int' to 'unsigned int' [-Werror,-Wsign-conversion]
  unsigned y = -1;
           ~   ^~
main.cpp:6:11: error: implicit conversion from 'double' to 'int' changes value from 9.300000000000001 to 9 [-Werror,-Wliteral-conversion]
  int z = 9.3;
      ~   ^~~
main.cpp:19:2: error: no newline at end of file [-Werror,-Wnewline-eof]
}
 ^
main.cpp:5:12: error: unused variable 'y' [-Werror,-Wunused-variable]
  unsigned y = -1;
           ^
main.cpp:4:7: error: unused variable 'x' [-Werror,-Wunused-variable]
  int x;
      ^
main.cpp:14:5: error: unannotated fall-through between switch labels [-Werror,-Wimplicit-fallthrough]
    case 10:
    ^
main.cpp:14:5: note: insert '[[clang::fallthrough]];' to silence this warning
    case 10:
    ^
    [[clang::fallthrough]]; 
main.cpp:14:5: note: insert 'break;' to avoid fall-through
    case 10:
    ^
    break; 
main.cpp:16:5: error: unannotated fall-through between switch labels [-Werror,-Wimplicit-fallthrough]
    default:
    ^
main.cpp:16:5: note: insert '[[clang::fallthrough]];' to silence this warning
    default:
    ^
    [[clang::fallthrough]]; 
main.cpp:16:5: note: insert 'break;' to avoid fall-through
    default:
    ^
    break; 
7 errors generated.
```
Скомпилируйте следующий код с флагами -Werror, -Wpedantic 
```C++
#include <iostream>

int main() {
  int n;
  std::cin >> n;
  int array[n];
}
```

Одна из ошибок
```C++
main.cpp:6:13: error: variable length arrays are a C99 feature [-Werror,-Wvla-extension]
  int array[n];
            ^
main.cpp:6:13: note: read of non-c
```

Итак, *напоминание с лекции*: размер выделенного на стеке массива, стоящий в квадратных скобках должен быть положительным
константным значением известным на этапе компиляции!

### Ограниченность C-style массивов и необходимость динамичеких

При работе с массивами, память для которых выделена на стеке, встречаемся со следующими проблемами:
1. Размер массива должен быть известен на этапе компиляции и не может меняться.
2. Размер массива ограничен размером стека (обычно это несколько мегабайт). Допустим, размер стека 8МБ, размер типа `int` 4 байта, тогда мы заведомо не сможем создать массив больше чем из $8 * 2^{20} / 4 = 2097152$ int'ов.
3. Массив перестанет существовать при выходе из соответствующей области видимости.

Все эти проблемы решают динамические массивы!

### Генерация случайных массивов

Сгенерировать случайное число можно при помощи функций `std::rand()` и `std::srand()`. Рассмотрим [пример](https://en.cppreference.com/w/cpp/numeric/random/rand) с cppreference.com

```C++
#include <cstdlib>
#include <iostream>
#include <ctime>
 
int main() 
{
    std::srand(std::time(nullptr)); // use current time as seed for random generator
    int random_variable = std::rand();
    std::cout << "Random value on [0 " << RAND_MAX << "]: " 
              << random_variable << '\n';
 
    // roll 6-sided dice 20 times
    for (int n=0; n != 20; ++n) {
        int x = 7;
        while(x > 6) 
            x = 1 + std::rand()/((RAND_MAX + 1u)/6);  // Note: 1+rand()%6 is biased
        std::cout << x << ' ';
    }
}
```

Функция `std::srand(<seed>)` фиксирует seed. Если передать при разных запусках программы одинаковый seed,
то функция `std::rand()` будет генерировать одну и ту же последовательность чисел. Это может понадобиться для дебага. В случае передачи  `std::time(nullptr)`(текущее время) seed будет разным при разных запусках программы.
