# Домашнее задание к занятию «3.2. Coroutines: Scopes, Cancellation, Supervision»

Выполненное задание прикрепите ссылкой на ваши GitHub-проекты в личном кабинете студента на сайте [netology.ru](https://netology.ru).

## Вопросы: Cancellation

### Вопрос №1

Отработает ли в этом коде строка `<--`? Поясните, почему да или нет.

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
    }
    delay(100)
    job.cancelAndJoin()
}
```
строка `<--` не отработает, 

так после создания корутины, сохранённой в переменной job,
спустя 100 милисекунд запускается метод cancelAndJoin(), 
который прерывает корутину, сохранённую в переменной job, 
и ждёт завершения этой прерванной карутины, 
а println("ok") должно было быть напечататано спустя полсекунды с помощью дочерней корутины, 
но она к этому моменту будет уже ждать завершение родительской корутины, 
а завершением родительской корутины, завершится и дочерняя.


### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        val child = launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
        delay(100)
        child.cancel()
    }
    delay(100)
    job.join()
}
```
строка `<--` не отработает, так как она должна запуститься спустя полсекунды,
но дочерняя корутина, в которой эта строка прописана, была прервана до этого из родитеской корутины.

## Вопросы: Exception Handling

### Вопрос №1

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    with(CoroutineScope(EmptyCoroutineContext)) {
        try {
            launch {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
строка `<--` не отработает, корутина запускается в блоке try/catch,
в этом случае исключение, созданное в карутине перехвачено не будет.

### Вопрос №2

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
строка `<--` отработает
coroutineScope отлавливает все исключения во всех дочерних корутинах и предоставляет их в родительской в виде исключения.

### Вопрос №3

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
строка `<--` отработает
supervisorScope позволяет завершать дочернюю корутину, в которой произошло исключение,
сохраняя активной родительску корутину.

### Вопрос №4

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    Thread.sleep(1000)
}
```
строка `<--` не отработает 
coroutineScope отлавливает все исключения во всех дочерних корутинах и предоставляет их в родительской в виде исключения.
Т.к. исключение в первой дочерней корутине должно произойти на пол секунды позже, чем во второй дочерней корутине,
то после того как произошло исключение во второй дочерней корутине, первая дочерняя корутина была отменена и её исключение не успело быть выбрашено.
(родительская корутина сохранила свое активное состояние)

### Вопрос №5

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
```
строка `<--` отработает
supervisorScope позволяет завершать дочернюю корутину, в которой произошло исключение, 
сохраняя активной родительску корутину и другие дочерние корутины.
В данном случае будет сначала отработает и завершится вторая дочерняя карутина, а потом первая дочерняя корутина

### Вопрос №6

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```
строка `<--` не отработает, т.к. исключение в родительской корутине будет выбрашено раньше по времени,
что приведёт к завершению дочерних корутин

### Вопрос №7

Отработает ли в этом коде строка `<--`. Поясните, почему да или нет.

```kotlin
fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext + SupervisorJob()).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
```
строка `<--` не отработает
SupervisorJob отменяет дочерние корутины при отмене родительской,
в данном случае родительской корутине было выбрашено исключение, и она завершилась,
что произошло на секунду раньше, чем должно была отработать строка `<--` в дочерней корутине.