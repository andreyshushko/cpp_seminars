## Переменные, выражения, циклы, условный оператор

### Тернарный оператор VS if-else

Тернарный оператор вместо блока if-else разумно использовать в двух случаях:
1. Рассмотрим две эквивалентные программы

```C++
#include <iostream>

int main() {
    int n;
    std::cin >> n;
    bool is_positive;
    
    if (n >= 0) {
        is_positive = true;
    } else {
        is_positive = false;
    }
}
```

```C++
#include <iostream>

int main() {
    int n;
    std::cin >> n;
    bool is_positive = (n >= 0) ? true : false;
}
```

Использование тернарного оператора сделало код более читаемым. 
Также мы сразу проинициализировали переменную is_positive (старайтесь не оставлять неинициализированные переменные в коде, если это возможно).

Итак, используйте тернарный оператор `<bool-expr> ? <expr1> : <expr2>`, если `<expr1>` и `<expr2>` состоят из одного короткого выражения и если возвращаемый тип будет очевиден.

2. Инициализация констант: 

`const bool is_positive = (n >= 0) ? true : false;`

### Оператор `swich`

Смотрим и разбираем [презентацию с лекции](https://docs.yandex.ru/docs/view?url=ya-disk-public%3A%2F%2FjjmCa5bP3BIA6WYDVc4Osk2mqQoUGaEz5lZgwh1AtVx5j8aMq8F3bfwCGrlzxFkEq%2FJ6bpmRyOJonT3VoXnDag%3D%3D%3A%2F02-Variables%26Expressions%26Statements%2FConditional.pdf&name=Conditional.pdf&nosw=1).

### Что такое lvalue, rvalue, prvalue
lvalue - это выражение, результат которого расположен в некотором постоянном месте в памяти. Например, переменные (именованые области памяти) - это lvalue.

Можно считать, что rvalue - это всё остальное, то есть это временные значения. Например, `5;`, `x++;`.

prvalue(pure rvalue)-выражения являются rvalue, но на них накладываются дополнительные ограничения. Пока о них мы говорить не будем.

### Различие `++x` и `x++`

С лекции: Префиксные операции `++` и `--` возвращают обновленное lvalue значение (то
есть сам аргумент). Постфиксные версии возвращают старое значение (до обновления) rvalue.

*Вопрос:* каковы возвращаемое значение и побочные эффекты выражения `++x++`?

*Ответ:* посмотрим на [приоритет операций в С++](http://cppstudio.com/post/302/). Поймём, что выражение эквивалентно 
`++(x++)`. Но `x++` возвращает rvalue, а префиксный инкремент принимает только lvalue. 
Получаем CE.

### *Дополнительно: pазличие `'\n'` и `std::endl`*

На [cppreference](https://en.cppreference.com/w/cpp/io/manip/endl) можно почитать, что делает `std::endl`.

### Домашнее задание

[Второй еженедельный контест](https://contest.yandex.ru/contest/40251/enter/)
