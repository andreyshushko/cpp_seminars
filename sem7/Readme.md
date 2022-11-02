### MergeSort

#### MergeSort

```C++
int* Copy(int* first_beg, int* first_end, int* second_beg){
    while(first_beg != first_end){
        *(second_beg++) = *(first_beg++);
    }
    return second_beg;
}

int* Merge(int* first_beg, int* first_end, int* second_beg, int* second_end, int* result){
    while(first_beg != first_end && second_beg != second_end){
        if(*first_beg <= *second_beg){
            *(result++) = *(first_beg++);
        } else {
            *(result++) = *(second_beg++);
        }
    }

    result = Copy(first_beg, first_end, result);
    result = Copy(second_beg, second_end, result);
    return result;
}

void MergeSortImpl(int* begin, int* end, int* result){
    size_t size = end - begin;
    if(size <= 1){
        return;
    }

    MergeSortImpl(begin,  begin + size / 2, result);
    MergeSortImpl(begin + size / 2, end, result);

    Merge(begin, begin + size / 2, begin + size / 2, end, result);

    Copy(result, result + size, begin);
}


void MergeSort(int* begin, int* end, int* result) {
  size_t size = end - begin;
  if (size <= 1) {
    return;
  }
  MergeSortImpl(begin, end, result);
}
```

#### InplaceMergeSort

Пока что не в терминах begin-end и без шаблонов (TBD)

```C++
void Swap(int& lhs, int& rhs){
    int tmp = lhs;
    lhs = rhs;
    rhs = tmp;
}

int* LowerBound(int* begin, int* end, int value){
    size_t size = end - begin;
    int* it;
    while(size > 0){
        it = begin;
        size_t step = size / 2;
        it += step;
        if(*it < value){
            begin = ++it;
            size -= step + 1;
        } else {
            size = step;
        }
    }
    return begin;
}

int* UpperBound(int* begin, int* end, int value){
    size_t size = end - begin;
    int* it;
    while(size > 0){
        it = begin;
        size_t step = size / 2;
        it += step;
        if(*it <= value){
            begin = ++it;
            size -= step + 1;
        } else {
            size = step;
        }
    }
    return begin;
}

void Reverse(int* begin, int* end){
    if(begin + 1 == end){
        return;
    }

    while(begin < end){
        Swap(*(begin++), *(--end));
    }
}

void Rotate(int* begin, int* mid, int* end){
    Reverse(begin, mid);
    Reverse(mid, end);
    Reverse(begin, end);
}  

void InplaceMerge(int* first_beg, int* first_end, int* second_beg, int* second_end){
    size_t first_size = first_end - first_beg;
    size_t second_size = second_end - second_beg;

     if(first_size == 0 || second_size == 0){
        return;
    }

    if(first_size == 1 && second_size == 1){
        if(*second_beg < *first_beg){
            Swap(*first_beg, *second_beg);
        }
        return;
    }

    int* b1_beg = first_beg;
    int* b1_end;
    int* b2_beg;
    int* b2_end = first_end;
    int* b3_beg = second_beg;
    int* b3_end; 
    int* b4_beg;
    int* b4_end = second_end;

    if(first_size >= second_size){
        b1_end = b1_beg + first_size / 2;
        b2_beg = b1_end;
        int* sep = LowerBound(b3_beg, b4_end, *b2_beg);
        b3_end = sep;
        b4_beg = sep;
    } else {
        b3_end = b3_beg + second_size / 2;
        b4_beg = b3_end;
        int* sep = UpperBound(b1_beg, b2_end, *b4_beg);
        b1_end = sep;
        b2_beg = sep;
    }

    Rotate(b2_beg, b2_end, b3_end); 
    size_t b3_size = b3_end - b3_beg;
    InplaceMerge(b1_beg, b1_end, b2_beg, b2_beg + b3_size);
    InplaceMerge(b2_beg + b3_size, b3_end, b4_beg, b4_end);

}


void InplaceMergeSort(int* begin, int* end){
    size_t size = end - begin;
    if(size <= 1){
        return;
    }
    
    InplaceMergeSort(begin, begin + size / 2);
    InplaceMergeSort(begin + size / 2, end);

    InplaceMerge(begin, begin + size / 2, begin + size / 2, end);
}
```

#### Почему нельзя писать реализацию шаблонных функций в .cpp файлах

Допустим, мы хотим написать модуль, содержащий функцию Swap. У нас есть три файла: main.cpp, swap.h, swap.cpp. Их содержимое приведено ниже.
```C++
// swap.h
#ifndef SWAP_H
#define SWAP_H

template<typename T>
void Swap(T& lhs, T& rhs);

#endif

//swap.cpp
#include "swap.h"

template<typename T>
void Swap(T& lhs, T& rhs) {
  T tmp = lhs;
  lhs = rhs;
  rhs = tmp;
}

//main.cpp
#include "swap.h"

int main() {
  int x = 3;
  int y = 4;
  Swap(x, y);
}
```

Данная программа скомпилируется, но упадёт на этапе линковки со следующей ошибкой
```bash
Undefined symbols for architecture x86_64:
  "void Swap<int>(int&, int&)", referenced from:
      _main in main-2ee172.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

Поймём, в чём проблема. В `main.cpp` присутсвует директива `#include`, которая на этапе препроцессинга подставляет в файл объявление функции `Swap`. Когда мы вызываем `Swap` в шестой строчке main'а, компилятор понимает, что подставить вместо `T` в объявление функции `Swap`. Затем отдельно происходит компиляция модуля, состоящего из `swap.cpp` и `swap.h`. Но здесь непонятно, как инстанцировать шаблон, кодогенерации не происходит, из-за чего у функции `Swap` нет определения и код падает с ошибкой линковки.

Решение: **помещать и объявление, и определение шаблонных функций в заголовочный файл**. 

#### Решения задач седьмого еженедельного задания

Лежат в этой же папке.

Запаковать файлы в архив можно при помощи команды [zip](https://linuxize.com/post/how-to-zip-files-and-directories-in-linux/).
