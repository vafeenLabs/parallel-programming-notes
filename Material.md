## Конспект лекций по параллелизму и многопоточности в C++ (подробно, без кода, в формате Markdown)

### Лекция 1: Введение в параллелизм (описание операций без кода)

*   **Запуск асинхронных задач:**
    *   При использовании `std::async` происходит запуск задачи в асинхронном режиме.
        *   **Операция:** Функция `std::async` принимает в качестве аргументов вызываемый объект (например, функцию) и его аргументы.
        *   **Процесс:** Система (среда исполнения) определяет, как именно будет выполняться задача. Это может быть создание нового потока, использование потока из пула потоков, или другие оптимизации.
        *   **Результат:** Функция `std::async` возвращает объект типа `std::future`. Этот объект представляет собой "обещание" получить результат работы асинхронной задачи в будущем.
    *   **Получение результата асинхронной задачи:**
        *   **Операция:** На объекте `std::future`, полученном от `std::async`, вызывается метод `get()`.
        *   **Процесс:** Метод `get()` приостанавливает выполнение текущего потока, пока асинхронная задача, связанная с этим `std::future`, не завершит свое выполнение и не вернет результат.
        *   **Результат:** Метод `get()` возвращает результат работы асинхронной задачи. Если задача сгенерировала исключение, то `get()` также сгенерирует исключение.
    *   **Ожидание завершения асинхронной задачи:**
        *   **Операция:** На объекте `std::future` вызывается метод `wait()`.
        *   **Процесс:** Метод `wait()` блокирует (приостанавливает) выполнение текущего потока до тех пор, пока асинхронная задача, связанная с этим `std::future`, не завершит свое выполнение.
        *   **Результат:** Метод `wait()` ничего не возвращает (`void`). Его единственная цель — дождаться завершения асинхронной задачи.
        *   **Автоматический вызов `wait()` в деструкторе:**
            *   **Процесс:** При уничтожении объекта `std::future` (когда он выходит из области видимости или удаляется), вызывается его деструктор.
            *   **Операция:** Внутри деструктора объекта `std::future` выполняется проверка. Если асинхронная задача еще не завершена (метод `wait()` еще не вызывался), то в деструкторе будет вызван метод `wait()`.
            *   **Результат:** Это действие гарантирует, что асинхронная задача будет выполнена до того, как `std::future` перестанет существовать, тем самым избегается потенциальная потеря результатов, утечки ресурсов или другие проблемы, связанные с незавершенной работой асинхронной задачи.

### Лекция 2: Параллелизм в вычислительных системах и управление потоками (операции без кода)

*   **2.1. Базовые операции управления потоками**
    *   **2.1.1. Запуск потока:**
        *   **Операция:** Создание объекта типа `std::thread`. При создании объекта `std::thread` в его конструктор передается вызываемый объект (функция, лямбда-выражение или функтор) и (опционально) аргументы, которые будут переданы в вызываемый объект.
        *   **Процесс:** Конструктор `std::thread` создает новый поток исполнения. Вызываемый объект начинает выполняться в этом новом потоке.
        *   **Результат:** Запускается новый поток, который выполняет заданную функцию параллельно с основным потоком программы.
    *   **2.1.2. Ожидание завершения потока:**
        *   **Операция:** На объекте `std::thread`, представляющем поток, вызывается метод `join()`.
        *   **Процесс:** Метод `join()` приостанавливает (блокирует) выполнение текущего (вызывающего) потока до тех пор, пока поток, для которого вызывается `join()`, не завершит свое выполнение.
        *   **Результат:** Вызывающий поток ожидает завершения вызываемого потока. После завершения вызываемого потока, управление возвращается в вызывающий поток. `join()` не возвращает никакого значения.
    *   **2.1.4. Запуск потоков в фоновом режиме:**
        *   **Операция:** Создание объекта `std::thread` и отказ от вызова метода `join()`.
        *    **Процесс:** Поток создается и начинает выполняться параллельно. Если `join()` не вызывается, то вызывающий поток не ждет завершения потока, и поток продолжает работать в "фоновом режиме", пока не завершится сам.
        *   **Результат:** Поток выполняется параллельно, но основной поток не ждет его завершения. Это может быть проблемой, если основной поток завершится раньше, чем фоновый поток, что может привести к проблемам с памятью или ресурсами. В этом случае используется `detach()`.
        *   **Операция:** На объекте `std::thread` вызывается метод `detach()`.
        *   **Процесс:** Метод `detach()` отсоединяет поток от объекта `std::thread`. После этого управление потоком и его временем жизни (life time) переходит к операционной системе.
        *   **Результат:** Поток продолжает выполняться в фоновом режиме и завершается, когда завершит свою работу, даже если программа завершилась (с некоторыми оговорками по ОС).
*   **2.3. Передача владения потоком:**
    *   **Операция:** Передача объекта `std::thread` через перемещение (move semantics), используя `std::move()` или в результате возвращения из функции.
    *   **Процесс:** Владение ресурсами потока (идентификатором, контекстом и любыми другими ассоциированными ресурсами) переходит к новому объекту `std::thread`.
    *   **Результат:** Старый объект `std::thread` теряет связь с потоком и становится "пустым" (не представляющим поток). Новый объект `std::thread` берет на себя ответственность за управление этим потоком.
*   **2.5. Идентификация потоков:**
    *   **Операция:** Вызов статической функции `std::this_thread::get_id()`.
    *   **Процесс:** Функция возвращает уникальный идентификатор текущего потока (то есть потока, из которого вызывается `std::this_thread::get_id()`).
    *   **Результат:** Функция возвращает значение типа `std::thread::id`, которое представляет уникальный идентификатор потока. Этот идентификатор может использоваться для сравнения потоков и для других целей, связанных с идентификацией потоков.

### Лекция 3: Разделение данных между потоками (операции без кода)

* **2.2. Передача аргументов функции потока:**
    *   **Операция:** Передача аргументов в конструктор объекта `std::thread` в момент создания потока.
    *   **Процесс:** Копии или ссылки на аргументы передаются в функцию, которая выполняется в новом потоке. При передаче по ссылке нужно соблюдать осторожность.
    *   **Результат:** Функция потока получает доступ к переданным данным.
*   **3.2. Защита разделяемых данных с помощью мьютексов**
    *   **3.2.1. Использование мьютексов в С++:**
        *   **Операция:** Создание объекта типа `std::mutex`.
        *   **Процесс:** Объект `std::mutex` выделяется и представляет мьютекс.
        *   **Результат:** Мьютекс готов к использованию для синхронизации.
        *   **Операция:** Вызов метода `lock()` на объекте `std::mutex`.
        *   **Процесс:** Если мьютекс не захвачен, вызывающий поток захватывает мьютекс, что позволяет ему эксклюзивно войти в защищенную секцию кода. Если мьютекс захвачен другим потоком, вызывающий поток приостанавливается (блокируется), пока мьютекс не будет освобожден.
        *   **Результат:** Вызывающий поток получает исключительный (эксклюзивный) доступ к защищаемым мьютексом данным.
        *   **Операция:** Вызов метода `unlock()` на объекте `std::mutex`.
        *   **Процесс:** Вызывающий поток освобождает мьютекс, что позволяет другим потокам, ожидающим этот мьютекс, продолжить свою работу.
        *   **Результат:** Другие потоки, ожидающие мьютекса, могут получить доступ к нему, и один из ожидающих потоков захватит мьютекс, как только он будет доступен.
    *   **3.2.2. Структурирование кода для защиты разделяемых данных:**
        *   **Операция:** Использование RAII-оберток (Resource Acquisition Is Initialization), таких как `std::lock_guard` или `std::unique_lock`, для работы с мьютексами.
        *   **Процесс:** RAII-обертка захватывает мьютекс в конструкторе и автоматически освобождает его в деструкторе.
        *   **Результат:** Обеспечивается автоматическое освобождение мьютекса даже в случае исключений, что предотвращает взаимоблокировки и другие проблемы, связанные с ручным управлением мьютексами.

## Лекция 4: Разделение данных между потоками

**Повторение:**

*   **Указатели на функции:**
    *   Объяснение: Указатели на функции - это переменные, которые хранят адрес функции в памяти. Это позволяет вызывать функции через указатель, что обеспечивает гибкость и динамизм.
    *   Операция: Объявление указателя на функцию (`return_type (*ptr_name)(arg_type1, arg_type2, ...)`) и присваивание ему адреса функции.
    *   Применение: Используется для передачи функций в качестве аргументов, для реализации обратных вызовов и для создания таблиц функций.

*   **Указатель на функцию с параметрами:**
    *   Объяснение: Как и обычный указатель на функцию, но явно описываются типы принимаемых параметров.
    *   Операция: При объявлении указателя, нужно указывать типы принимаемых параметров, например `void (*ptr_name)(int, double)`.
    *   Применение: Обеспечивает типобезопасность при вызове функции через указатель.

*   **Шаблон `std::function`:**
    *   Объяснение: Шаблон `std::function` является универсальной оберткой для любого вызываемого объекта (функции, лямбда-выражения, функторы). Он позволяет хранить и вызывать любой вызываемый объект с совместимой сигнатурой.
    *   Операция: Объявление переменной типа `std::function<return_type(arg_type1, arg_type2, ...)>`, которому можно присвоить любой подходящий вызываемый объект.
    *   Применение: Обеспечивает возможность работать с разными типами вызываемых объектов через общий интерфейс. Повышает гибкость и абстракцию.

*   **`std::bind` для передачи заказчику с параметрами:**
    *   Объяснение: `std::bind` позволяет создавать новый вызываемый объект, связывая (фиксируя) некоторые аргументы исходного вызываемого объекта.
    *   Операция: Вызов `std::bind(callable, arg1, arg2, ...)` с указанием вызываемого объекта и фиксированных аргументов.
    *   Применение: Используется для создания кастомных вызовов функций с определенными параметрами и для передачи в качестве аргумента в `std::function`.

*   **`std::bind` для перевода указателя на `std::placeholder::_x`:**
    *   Объяснение: `std::placeholder::_1`, `std::placeholder::_2`, и т.д., используются в `std::bind` для обозначения позиций аргументов, которые будут переданы в момент вызова.
    *   Операция: Использование `std::placeholders` в `std::bind`, например: `std::bind(func, std::placeholders::_1, 5)` создает функтор, принимающий 1 параметр.
    *   Применение: Позволяет задать порядок аргументов и частично применять функции, подготавливая их к последующему вызову.

*   **Правило ноля, правило пяти:**
    *   **Правило Ноля:** Если класс не управляет какими-либо ресурсами (не имеет указателей на динамическую память, дескрипторов файлов и т.д.), то не нужно определять ни деструктор, ни конструктор копирования, ни оператор присваивания копированием, ни конструктор перемещения, ни оператор присваивания перемещением. Компилятор сгенерирует их автоматически.
    *   **Правило Пяти:** Если класс управляет какими-либо ресурсами, то необходимо явно определить (или удалить) деструктор, конструктор копирования, оператор присваивания копированием, конструктор перемещения и оператор присваивания перемещением.
    *   Объяснение:  Эти правила помогают гарантировать правильное управление ресурсами и предотвратить утечки памяти или проблемы с двойным освобождением.

**Новые темы:**

*   **3.2.4. Взаимоблокировка: проблема и решение:**
    *   Объяснение: Взаимоблокировка (deadlock) — это ситуация, когда два или более потоков оказываются заблокированными, ожидая ресурсов, которые заняты другими потоками в этой же группе.
    *   Причины: Часто возникает при циклическом ожидании мьютексов.
    *   Решение: Простые решения включают в себя использование `std::lock`, `std::try_lock`, и избегание захвата нескольких мьютексов одновременно. Захват мьютексов в одном и том же порядке разными потоками. Более сложные, использование иерархий мьютексов.

*   **3.2.5. Дополнительные рекомендации, как избежать взаимоблокировок:**
    *   Объяснение:
        *   Избегать излишней блокировки: Блокировать только на время, необходимое для доступа к общим данным.
        *   Использовать тайм-ауты: `std::timed_mutex` для ограничения времени ожидания мьютекса.
        *   Минимизировать время блокировки: Код внутри критической секции должен выполняться быстро и не содержать потенциальных блокирующих операций.

*   **3.2.6. Ошибка блокировки с помощью `std::unique_lock`:**
    *   Объяснение:  `std::unique_lock` является гибкой оберткой для мьютексов, но может привести к ошибке, если используется некорректно.
        *   Ошибка: Например, попытка захвата мьютекса, когда он уже захвачен этим же потоком (если не использовался `std::recursive_mutex`).
    *   Решение: Использовать `std::recursive_mutex` для рекурсивной блокировки или убедиться, что мьютекс не блокируется повторно.

*   **3.2.7. Передача владения мьютексом между контекстами:**
    *   Объяснение:  `std::unique_lock` можно использовать для передачи владения мьютексом между разными контекстами (например, из одного потока в другой).
    *   Операция: Использование `std::move()` для перемещения `std::unique_lock`.
    *   Применение: Это полезно, когда мьютекс нужен в разных функциях или потоках.

*   **3.2.8. Выбор правильной гранулярности блокировки:**
    *   Объяснение: Гранулярность блокировки — это размер критической секции, защищенной мьютексом.
    *   Рекомендации:
        *   Мелкая гранулярность (блокировка меньших участков кода) увеличивает параллелизм, но может повысить накладные расходы.
        *   Крупная гранулярность (блокировка больших участков кода) снижает параллелизм, но уменьшает накладные расходы.
    *   Вывод: Выбор правильной гранулярности зависит от конкретной задачи и компромисса между производительностью и безопасностью.

## Лекция 5: Разделение данных между потоками

**Повторение:**

*   Основные моменты из предыдущих лекций:
    *   Потоки и параллелизм
    *   Проблемы разделения данных (гонки)
    *   Мьютексы и их использование
    *   Проблемы взаимоблокировок и способы их предотвращения.
    *  Правила ноля и пяти
    * Указатели на функции
    * Шаблоны функций и std::bind

**Новые темы:**

*   **3.3. Другие средства защиты разделяемых данных:**
    *   Объяснение: Кроме мьютексов, существуют и другие инструменты для обеспечения потокобезопасности.

*   **3.3.1. Защита разделяемых данных во время инициализации:**
    *   Объяснение:  Защита данных во время инициализации, когда несколько потоков могут одновременно пытаться инициализировать данные.
    *   Решение:  Использование `std::call_once` для гарантии, что инициализация будет выполнена только один раз.

*   **3.3.2. Защита редко обновляемых структур данных:**
    *   Объяснение:  Защита структур данных, которые редко обновляются, но часто читаются.
    *   Решение:  Использование `std::shared_mutex` (читательско-писательская блокировка) для повышения параллелизма при чтении.

*   **3.3.3. Рекурсивная блокировка:**
    *   Объяснение:  Рекурсивная блокировка, когда один поток может захватить один и тот же мьютекс несколько раз.
    *   Решение:  Использование `std::recursive_mutex`, который позволяет одному потоку многократно захватывать мьютекс без взаимоблокировки.

## Лекция 6: Синхронизация потоков

**ГЛАВА 4. Синхронизация параллельных операций**

*   **4.1. Ожидание события или иного условия:**
    *   Объяснение:  Механизмы для потоков, которые ожидают наступления определенного события или выполнения определенного условия.
    *   **4.1.1. Ожидание условия с помощью условных переменных:**
        *   Объяснение: Условные переменные позволяют потокам ожидать выполнения определенного условия, и не тратить процессорное время.
        *   Использование:  `std::condition_variable` для ожидания наступления условия. `wait()` блокирует поток, `notify_one()` пробуждает один ожидающий поток, `notify_all()` пробуждает все ожидающие потоки.
        *   Механизм работы: Условные переменные должны использоваться вместе с мьютексами для защиты общих данных.
    *   **4.1.2. Потокобезопасная очередь на базе условных переменных:**
        *   Объяснение: Применение условных переменных для создания потокобезопасной очереди.
        *   Принцип работы: Потоки-производители помещают данные в очередь и уведомляют потоки-потребители. Потоки-потребители ждут появления данных в очереди, используя условные переменные.
        *   Применение: Используется для организации асинхронной передачи данных между потоками.

## Лекция 7: Управление потоками в WinAPI

*   **1. Дескрипторы (ручки), `LPCVOID`, `LPVOID` и другие типы данных WinAPI:**
    *   Объяснение:
        *   Дескрипторы: Представляют собой абстрактные идентификаторы для различных системных объектов (потоков, файлов, событий и т.д.).
        *   `LPCVOID`:  Константный указатель на void (generic pointer to data).
        *   `LPVOID`:  Указатель на void (generic pointer to data).
        *   Другие типы данных WinAPI:  `DWORD` (unsigned long), `HANDLE` (дескриптор).

*   **2. Управление потоками в WinAPI:**
    *   **2.1. Создание потока:**
        *   Операция: Использование функции `CreateThread`.
        *   Аргументы: Атрибуты безопасности, размер стека, указатель на функцию потока, аргумент для функции потока, флаги создания.
        *   Результат: Создается новый поток, который начинает выполнять заданную функцию. Возвращается дескриптор потока.
    *   **2.2. Приостановка потока:**
        *   Операция: Использование функции `SuspendThread`.
        *   Аргументы: Дескриптор потока, который нужно приостановить.
        *   Результат: Указанный поток приостанавливает выполнение. Поток можно будет возобновить с помощью `ResumeThread`.
    *   **2.3. Завершение потока:**
        *   Операция: Использование функции `TerminateThread`.
        *   Аргументы: Дескриптор потока и код возврата.
        *   Результат: Поток завершается насильственно, что может привести к проблемам с ресурсами. Рекомендуется использовать завершение потока через его функцию (а не `TerminateThread`).

## Лекция 8: Управление потоками в WinAPI

*   **Повторение:**
    *   Размеры базовых типов данных (int, char, float и т.д.) и выравнивание структур: Объяснение, как размеры типов данных и выравнивание влияют на размер структур и как это может влиять на производительность.

*   **`GetLastError()` и поиск ошибок:**
    *   Объяснение:  Функция `GetLastError()` возвращает код ошибки последней вызванной функции WinAPI.
    *   Использование: Понимание, что эта функция необходима для диагностики ошибок в Windows. Важность понимания кода ошибки для отладки.

*   **3. Синхронизация потоков:**
    *   **3.1. Ожидание завершения потока:**
        *   Операция: Использование функции `WaitForSingleObject`.
        *   Аргументы: Дескриптор потока, тайм-аут.
        *   Результат: Вызывающий поток ожидает завершения указанного потока.
    *   **3.2. Объект ядра Событие:**
        *   Объяснение:  Событие (Event) — это объект ядра для синхронизации, который может находиться в сигнальном или несигнальном состоянии.
        *   Операция: `CreateEvent` для создания события, `SetEvent` для перевода в сигнальное состояние, `ResetEvent` для перевода в несигнальное, `WaitForSingleObject` для ожидания.
        *   Применение: Потоки могут ожидать, когда событие станет сигнальным, для продолжения работы.
    *   **3.3. Объект ядра Мьютекс:**
        *   Объяснение:  Мьютекс — это объект ядра для синхронизации, который позволяет одному потоку захватить ресурс.
        *   Операция: `CreateMutex` для создания мьютекса, `WaitForSingleObject` для захвата мьютекса, `ReleaseMutex` для освобождения мьютекса.
        *   Применение: Используется для защиты общих ресурсов от одновременного доступа разных потоков.

## Лекция 9: Управление потоками в WinAPI

*   **3.4. Объект ядра Семафор:**
    *   Объяснение:  Семафор — это объект ядра для синхронизации, который позволяет нескольким потокам получать доступ к ресурсу ограниченное число раз.
    *   Операция:  `CreateSemaphore` для создания семафора, `WaitForSingleObject` для ожидания, `ReleaseSemaphore` для освобождения.
    *   Применение:  Используется для управления доступом к пулу ресурсов.
*   **3.5. Критические секции:**
    *   Объяснение:  Критические секции — это механизмы синхронизации, которые быстрее мьютексов, но ограничены использованием в одном процессе.
    *   Операция: `InitializeCriticalSection` для инициализации, `EnterCriticalSection` для входа, `LeaveCriticalSection` для выхода, `DeleteCriticalSection` для удаления.
    *   Применение: Используются для защиты общих ресурсов, когда мьютексы не требуются.
*   **Добавление `CreateThread` на практике:**
    *   Объяснение:  Разбор конкретных примеров создания потоков с использованием `CreateThread`
    *  Применение:  Наглядный пример, как использовать функцию `CreateThread` на практике.

*   **Утверждение (assert) через `ASSERT`, `STATIC_ASSERT`:**
    *   Объяснение:
        *   `ASSERT`:  Проверяет условие во время выполнения, и, если оно ложно, прерывает программу (в отладочной версии).
        *   `STATIC_ASSERT`:  Проверяет условие во время компиляции, если условие ложно - выдает ошибку компиляции.
    *   Применение:  Используется для проверки условий, которые должны быть истинными для корректной работы программы.

## Лекция 10: Управление потоками в Linux

**Повторение:**

*   **Директивы препроцессора сообщения `#pragma message`, `#error`, `#warning`:**
    *   Объяснение: `#pragma message`  выводит сообщение во время компиляции. `#error` вызывает ошибку во время компиляции. `#warning` вызывает предупреждение во время компиляции.
    *   Применение: Используются для информирования разработчиков, выявления ошибок или предупреждений.
*   **Ошибка во многопоточных приложениях:**
    *   Объяснение: Повторение типичных ошибок в многопоточных приложениях (гонки, взаимоблокировки).

*   **1. Создание потока:**
    *   Операция: Использование функции `pthread_create`.
    *   Аргументы: Указатель на структуру атрибутов, указатель на идентификатор нового потока, указатель на функцию, аргумент функции.
    *   Результат: Создает новый поток, который начинает выполнение переданной функции.
*   **2. Завершение потока:**
    *   Объяснение: Потоки в Linux, как правило, завершаются естественным образом, когда завершается функция, с которой поток был создан.
    *   Операция: Завершение функции потока.
    *   Результат: Поток завершается и освобождает ресурсы.
*   **3. Ожидание потока:**
     *  Операция: Использование функции `pthread_join`.
     *  Аргументы: Идентификатор потока, возвращаемое значение.
     *  Результат: Вызывающий поток ожидает завершения указанного потока.
*   **4. Досрочное завершение потока:**
    *   Операция: Использование функции `pthread_cancel`.
    *   Аргументы: Идентификатор потока, который нужно отменить.
    *   Результат: Указанный поток завершается.
*   **5. Отсоединение потока:**
    *   Операция: Использование функции `pthread_detach`.
    *   Аргументы: Идентификатор отсоединяемого потока.
    *   Результат:  Отсоединяет поток от вызывающего потока, что позволяет системе самостоятельно управлять ресурсами завершившегося потока.
*   **6. Синхронизация потоков:**
    *   **6.1. Мьютекс `pthread_mutex_t`:**
        *   Операция:  `pthread_mutex_init` для инициализации мьютекса, `pthread_mutex_lock` для захвата, `pthread_mutex_unlock` для освобождения, `pthread_mutex_destroy` для уничтожения.
        *   Применение:  Используется для защиты общих данных от одновременного доступа нескольких потоков.

## Лекция 11: Управление приложениями в Linux

*   **6.2. Семафоры `sem_init`:**
    *   Операция:  `sem_init` для инициализации семафора, `sem_wait` для ожидания, `sem_post` для освобождения.
    *   Применение:  Используется для управления доступом к пулу ресурсов.
*   **6.3. Условная переменная `pthread_cond_timedwait`:**
    *   Операция: `pthread_cond_init` для инициализации, `pthread_cond_wait` для ожидания, `pthread_cond_signal` для уведомления одного потока, `pthread_cond_broadcast` для уведомления всех потоков.
    *   Применение:  Используется для организации ожидания наступления условия.
*   **6.4. Ожидание потока `pthread_join`:**
     *  Операция: Использование функции `pthread_join`.
     *  Аргументы: Идентификатор потока, возвращаемое значение.
     *  Результат: Вызывающий поток ожидает завершения указанного потока.
*   **6.5. Барьеры `pthread_barrier_init`:**
    *   Операция: `pthread_barrier_init` для инициализации барьера, `pthread_barrier_wait` для ожидания.
    *   Применение:  Используется для синхронизации нескольких потоков, чтобы все они дождались друг друга в определенной точке программы.
*   **6.6. Spin-блокировки `pthread_spin_init`:**
    *   Операция: `pthread_spin_init` для инициализации, `pthread_spin_lock` для захвата, `pthread_spin_unlock` для освобождения.
    *   Применение: Используются в ситуациях, когда блокировка кратковременна.
*   **7. Потоки versus процессы:**
    *   Объяснение: Сравнение потоков и процессов, их различия в адресных пространствах, ресурсах, накладных расходах. Когда стоит использовать потоки, а когда процессы.