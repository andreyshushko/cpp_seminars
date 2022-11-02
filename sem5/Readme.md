## Рекурсия, сортировки

### Бинарный поиск через рекурсию

Вспомним нерекурсивную реализацию.

```C++
int BinarySearchLeft(const int* arr, const int arr_size, const int key) {
  int left_board = -1;
  int right_board = arr_size;
  int middle;

  while (left_board < right_board - 1) {
    middle = left_board + (right_board - left_board) / 2;
    if (arr[middle] < key) {
      left_board = middle;
    } else {
      right_board = middle;
    }
  }

  return right_board;
}
```

На её основе напишем рекурсивную.

```C++

int BinarySearchLeftRecursiveImpl(const int* arr, const int size, const int key, const int left, const int right) {
    int middle = left + (right - left) / 2;

    if (left >= right - 1) {
      return right;
    }

    if (key > arr[middle]) {
      return BinarySearchLeftRecursiveImpl(arr, size, key, middle, right);
    }

    return BinarySearchLeftRecursiveImpl(arr, size, key, left, middle);
}

int BinarySearchLeftRecursive(const int* arr, const int size, const int key) {
  return BinarySearchLeftRecursiveImpl(arr, size, key, -1, size);
}
```
### Ошибка Stack overflow

При вызове функции на стек кладётся адрес возврата. Размер стека ограничен (обычно порядка единиц мегабайт). 
Таким образом, если постоянно заходить в функции и не возвращаться из них, то стек переполнится и возникнет ошибка StackOverflow.

### Квадратичные сортировки

1. Сортировка выбором
```C++
/** Lhs/rhs is left/right-hand side
 * Let's use these parameter names in swap function, comparators, and similar taking two arguments **/
void Swap(int& lhs, int& rhs) {
  int tmp = lhs;
  lhs = rhs;
  rhs = tmp;
}

void SelectionSort(int* arr, const int size) {
  for (int i = 0; i < size - 1; ++i) {
    int idxMin = i;
    for (int j = i + 1; j < size; ++j) {
      if (arr[j] < arr[idxMin]) {
        idxMin = j;
      }
    }
    if (idxMin != i) {
      Swap(arr[i], arr[idxMin]);
    }
  }
}
```
2. Сортировка вставками
```C++
void InsertionSort(int* arr, const int size) {
  for (int i = 1; i < size; ++i) {
    for (int j = i; j > 0 && arr[j] < arr[j - 1]; --j) {
      Swap(arr[j], arr[j - 1]);
    }
  }
}
```
3. Сортировка пузырьком
Вначале напишем наивную реализацию, затем добавим небольшую оптимизацию: если при очередном "проходе" по массиву мы не передвинули ни один элемент, то массив уже отсортирован.

```C++
void BubbleSortNaive(int* arr, const int size) {
  for (int i = 0; i < size - 1; ++i) {
    for (int j = 0; j < size - i - 1; ++j) { // why can we use size - i - 1?
      if (arr[j] > arr[j + 1]) {
        Swap(arr[j], arr[j + 1]);
      }
    }
  }
}

void BubbleSort(int* arr, const int size) {
  for (int i = 0; i < size - 1; ++i) {
    bool werePermutations = false;
    for (int j = 0; j < size - i - 1; ++j) {
      if (arr[j] > arr[j + 1]) {
        Swap(arr[j], arr[j + 1]);
        werePermutations = true;
      }
    }
    if (!werePermutations) {
      return;
    }
  }
}
```

### Сортировка подсчётом
В этой реализации все элементы должны быть строго меньше `max`.

```C++
int CountingSort(int* arr, const int size, int max) {
  int c[max];
  for (int i = 0; i < max; ++i) {
    c[i] = 0;
  }
  for (int i = 0; i < size; ++i) {
    ++c[arr[i]];
  }
  int pos = 0;
  for (int j = 0; j < max; ++j) {
    for (int i = 0; i < c[j]; ++i) {
      arr[pos] = j;
      ++pos;
    }
  }
}

```
