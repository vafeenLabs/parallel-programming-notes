# std::async 
Запускает асинхронную задачу с получением результата в будущем. Возвращает **std::future**, значение из которого получается с помощью метода **get()**
Параметры ссылки передаются в **async** через **std::ref**
Есть параметр **std::launch** и может принимать **std::launch::deffered** - отложить до вызова **wait()** или **get()** и может принимать **std::launch::async** - выполнение в отдельном потоке. 

# **std::packaged_task**
Оборачивает задачу, создавая связанный `std::future` для отслеживания результата.

# std::future  && std::shared_future 
Объект будущего результата
**shared_future** отличается тем что потоки могут работать с ним безо всякой синхронизации. 

# std::thread 
Для создания выполнения задачи в потоке нужно передать в конструктор **std::thread** первым аргументом указатель на задачу потока, а далее аргументы. 

# std::mutex 
Примитив синхронизации для организации критических секций. 
Вход в критическую секцию **mut.lock()**, выход - **mut.unlock()**

# volatile 
`volatile int& global_sum` - сообщает компилятору, что это значение может измениться извне: системой или другими потоками, и поэтому компилятор каждый раз загружает это значение из памяти. 

# std::lock_guard 
Данный примитив синхронизации захватывает мьютекс в конструкторе и освобождает в деструкторе, гарантируя, что мьютекс будет освобожден. 

# join 
Это функция завершения потока, после вызова которой поток уже не будет ассоциироваться с его переменной
(перед вызовом лучше проверять **joinable()**)

# std::condition_variable 
Переменная для блокирования одного или нескольких потоков до момента поступления сигнала от другого потока о выполнении некоторого условия. 
(Используются вместе с мьютексом)
Эта переменная позволяет разблокировать один **notify_one()** или все **notify_all()** потоки для выполнения задач, в то время как поток ждет наступления события с помощью **wait()**

# wait(мьютекс, условие: лямбда)
Поток захватывает мьютекс с помощью **unique_lock** внутри **wait** и проверяет условие. Если условие выполнено, **wait** передаст управление потоку, и мьютекс будет захвачен, а потом по выполнению таска освобожден. 
# std::unique_lock. 
Метод блокирует поток и добавляет его в очередь потоков, ожидающих сигнала от условной переменной; поток пробуждается, когда будет получен сигнал от условной переменной или в случае ложного пробуждения.