# SingleLinkedList-
Односвязный список. Для него написал последовательный итератор (Forward itertor). Обеспечена строгая гарантия безопасности.
## Описание
Некоторые особенности данного поисковика:
* Реализован конструктор по умолчанию, который создаёт пустой список.
* Реализован метод `GetSize`, который возвращает количество элементов в списке.
* Реализован метод `IsEmpty`, который возвращает `true`, если список пустой, и `false` в противном случае.
* Реализован метод `PushFront`, который делает вставки элемента в начало односвязного списка.
* Реализован метод `Clear`, который очищает список и не выбрасывает исключений. 
* При разрушении списка удаляются все его элементы.
* Реализована поддержка перебора элементов контейнера `SingleLinkedList` с помощью шаблонного класса `BasicIterator`, на основе которого объявлены константный и неконстантный итераторы списка.
* Реализованы операции сравнения `==`, `!=`, `<`, `>`, `<=`, `>=`.
* Реализован обмен содержимого двух списков с использованием метода `swap` и шаблонной функции `swap`.
* Реализовано конструирование односвязного списка на основе `initializer_list`.
* Реализованы надёжные конструктор копирования и операция присваивания (операция присваивания обеспечивает строгую гарантию безопасности исключений - если в процессе присваивания будет выброшено исключение, содержимое левого аргумента операции присваивания должно остаться без изменений).
* Реализован метод `PopFront`, который удаляет первый элемента непустого списка за время O(1) (не выбрасывает исключений).
* Реализован метод `InsertAfter` - за время O(1) вставляет в список новое значение следом за элементом, на который ссылается переданный в `InsertAfter` итератор (метод обеспечивает строгую гарантию безопасности исключений).
* Реализован метод `EraseAfter` - за время O(1) удаляет из списка элемент, следующий за элементом, на который ссылается переданный в `InsertAfter` итератор (не выбрасывает исключений).
* Реализованы методы `before_begin` и `cbefore_begin` - возвращают итераторы, ссылающиеся на фиктивную позицию перед первым элементом списка. Такой итератор используется как параметр для методов `InsertAfter` и `EraseAfter`, когда нужно вставить или удалить элемент в начале списка (разыменовывать этот итератор нельзя).
## Инструкция по использованию
Перенесите файл в свой проект и используйте `SingleLinkedList` как замену стандартному `list`.
> Пример использования и тесты, проверяющие работоспособность класса:  
```c++
void Test4() {
    struct DeletionSpy {
        ~DeletionSpy() {
            if (deletion_counter_ptr) {
                ++(*deletion_counter_ptr);
            }
        }
        int* deletion_counter_ptr = nullptr;
    };

    // Проверка PopFront
    {
        SingleLinkedList<int> numbers{ 3, 14, 15, 92, 6 };
        numbers.PopFront();
        assert((numbers == SingleLinkedList<int>{14, 15, 92, 6}));

        SingleLinkedList<DeletionSpy> list;
        list.PushFront(DeletionSpy{});
        int deletion_counter = 0;
        list.begin()->deletion_counter_ptr = &deletion_counter;
        assert(deletion_counter == 0);
        list.PopFront();
        assert(deletion_counter == 1);
    }

    // Доступ к позиции, предшествующей begin
    {
        SingleLinkedList<int> empty_list;
        const auto& const_empty_list = empty_list;
        assert(empty_list.before_begin() == empty_list.cbefore_begin());
        assert(++empty_list.before_begin() == empty_list.begin());
        assert(++empty_list.cbefore_begin() == const_empty_list.begin());

        SingleLinkedList<int> numbers{ 1, 2, 3, 4 };
        const auto& const_numbers = numbers;
        assert(numbers.before_begin() == numbers.cbefore_begin());
        assert(++numbers.before_begin() == numbers.begin());
        assert(++numbers.cbefore_begin() == const_numbers.begin());
    }

    // Вставка элемента после указанной позиции
    {  // Вставка в пустой список
        {
            SingleLinkedList<int> lst;
            const auto inserted_item_pos = lst.InsertAfter(lst.before_begin(), 123);
            assert((lst == SingleLinkedList<int>{123}));
            assert(inserted_item_pos == lst.begin());
            assert(*inserted_item_pos == 123);
        }

        // Вставка в непустой список
        {
            SingleLinkedList<int> lst{ 1, 2, 3 };
            auto inserted_item_pos = lst.InsertAfter(lst.before_begin(), 123);

            assert(inserted_item_pos == lst.begin());
            assert(inserted_item_pos != lst.end());
            assert(*inserted_item_pos == 123);
            assert((lst == SingleLinkedList<int>{123, 1, 2, 3}));

            inserted_item_pos = lst.InsertAfter(lst.begin(), 555);
            assert(++SingleLinkedList<int>::Iterator(lst.begin()) == inserted_item_pos);
            assert(*inserted_item_pos == 555);
            assert((lst == SingleLinkedList<int>{123, 555, 1, 2, 3}));
        };
    }

    // Вспомогательный класс, бросающий исключение после создания N-копии
    struct ThrowOnCopy {
        ThrowOnCopy() = default;
        explicit ThrowOnCopy(int& copy_counter) noexcept
            : countdown_ptr(&copy_counter) {
        }
        ThrowOnCopy(const ThrowOnCopy& other)
            : countdown_ptr(other.countdown_ptr)  //
        {
            if (countdown_ptr) {
                if (*countdown_ptr == 0) {
                    throw std::bad_alloc();
                }
                else {
                    --(*countdown_ptr);
                }
            }
        }
        // Присваивание элементов этого типа не требуется
        ThrowOnCopy& operator=(const ThrowOnCopy& rhs) = delete;
        // Адрес счётчика обратного отсчёта. Если не равен nullptr, то уменьшается при каждом копировании.
        // Как только обнулится, конструктор копирования выбросит исключение
        int* countdown_ptr = nullptr;
    };

    // Проверка обеспечения строгой гарантии безопасности исключений
    {
        bool exception_was_thrown = false;
        for (int max_copy_counter = 10; max_copy_counter >= 0; --max_copy_counter) {
            SingleLinkedList<ThrowOnCopy> list{ ThrowOnCopy{}, ThrowOnCopy{}, ThrowOnCopy{} };
            try {
                int copy_counter = max_copy_counter;
                list.InsertAfter(list.cbegin(), ThrowOnCopy(copy_counter));
                assert(list.GetSize() == 4u);
            }
            catch (const std::bad_alloc&) {
                exception_was_thrown = true;
                assert(list.GetSize() == 3u);
                break;
            }
        }
        assert(exception_was_thrown);
    }

    // Удаление элементов после указанной позиции
    {
        {
            SingleLinkedList<int> lst{ 1, 2, 3, 4 };
            const auto& const_lst = lst;
            const auto item_after_erased = lst.EraseAfter(const_lst.cbefore_begin());
            assert((lst == SingleLinkedList<int>{2, 3, 4}));
            assert(item_after_erased == lst.begin());
        }
        {
            SingleLinkedList<int> lst{ 1, 2, 3, 4 };
            const auto item_after_erased = lst.EraseAfter(lst.cbegin());
            assert((lst == SingleLinkedList<int>{1, 3, 4}));
            assert(item_after_erased == (++lst.begin()));
        }
        {
            SingleLinkedList<int> lst{ 1, 2, 3, 4 };
            const auto item_after_erased = lst.EraseAfter(++(++lst.cbegin()));
            assert((lst == SingleLinkedList<int>{1, 2, 3}));
            assert(item_after_erased == lst.end());
        }
        {
            SingleLinkedList<DeletionSpy> list{ DeletionSpy{}, DeletionSpy{}, DeletionSpy{} };
            auto after_begin = ++list.begin();
            int deletion_counter = 0;
            after_begin->deletion_counter_ptr = &deletion_counter;
            assert(deletion_counter == 0u);
            list.EraseAfter(list.cbegin());
            assert(deletion_counter == 1u);
        }
    }
}
```
## Системные требования
- С++17 (C++1z)
***
![giffif](https://user-images.githubusercontent.com/93004994/164434944-d2e29257-6f92-4aae-a542-ecb36bd52df1.gif)
