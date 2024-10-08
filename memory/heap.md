Куча - тип управления памятью в котором данные хранятся без определённого порядка и могут быть использованы из любой точки программы

# Адрес
Проще всего работу с памятью объяснить на примере с улицей с почтовыми ящиками.
Представим улицу, где дома стоят на одной стороне. На улице 128 домов и у каждого есть почтовый ящик с флажком 📫

В момент времени флажок может быть либо поднят (значение 1) либо опущен (значение 0)
Для того чтобы придать какой-то смысл примеру, скажем что нужно доставить письмо Александру. Александр живёт в доме номер 57. Значит мы пойдём к ящику (ячейке) 57 и засунем в него письмо и установим флажок (присвоим 1)

Точно так же процессор работает с памятью (не только в куче но и в целом)

Память имеет физический адрес выражаемый в виде целого положительного числа. Фактически этот адрес это не номер ячейки, а смещение относительно начала памяти. Т.е при обращении в 57ю ячейку мы не ищем ячейку с номером 57, мы пропускаем 57 ячеек от начала улицы и кладём единицу.

> Куча (или heap) как раз представляет собой такую улицу на которой живёт огромное количество ячеек. 

# Указатель

Основным преимуществом такого подхода является тот факт, что мы можем из любой точки программы обратиться к этому участку памяти и взаимодействовать с ним, зная только его адрес

Для того чтобы получить адрес чего либо в Go используется инструкция "&" 

```go
package main

func main() {
  a := int8(6)
  b := &a 
  
  _ = b
}
```

В приведённом примере создаётся переменная "a" и переменная "b". В первой переменной лежит некоторое значение с типом int (6), а во второй переменной лежит указатель, на участок памяти (адрес), где лежит значение с типом `int8`

Переменная "b" будет иметь тип `*int8`. При описании типов символ `*` обозначает не сам тип, а указатель на память, в которой лежит примитив/объект с таким типом.

Мы не можем просто сказать что "b"  это ссылка без типа (на самом деле можем через unsafe), так как указатель - это всего лишь адрес, нам **важно** знать сколько ещё байт (точнее машинных слов) начиная с этого адреса нужно прочитать чтобы получить полную картину.

В данном случае пускай переменная "a" лежит по адресу 5 (т.е от начал памяти нужно пропустить 5 байт чтобы дойти до начала куска памяти в котором лежит "a")
Выглядит это примерно так:

```
0000_0000110_0000...
```

> Для наглядности участок памяти отвечающих за "а" выделен подчёркиванием. в действительности вся память выглядит как идущие подряд нули и единицы

Вот сам по себе адрес - 5 не говорит ничего о данных, которые по нему находятся. Однако в купе с типом `int8` мы (компилятор) понимает что начиная с 5го бита нужно прочитать ещё 8 чтобы получить нужное значение.

Естественно из за того что за нас это делает компилятор (точнее сказать среда выполнения) мы платим скоростью выполнения, но в реальности удобство которое мы получаем от использования связки указатель+тип **значительно** превосходит затраты на работу среды.

Однако сама по себе "b" тоже занимает место в памяти, так как указывает на другое место в памяти (5), а значит в полном виде куча выглядит вот так

```
0000_0000110_00000101_...
    ^ а=6   ^ b=5     
```

Если вывести значения переменных в консоль следующим образом
```go
package main

func main() {
	a := int8(6)
	b := &a 
  
	fmt.Println(a)
	fmt.Println(b)
}
```

Получим примерно такой вывод:
```
| 6
| 0xc00001206d
```

"Примерно" такой потому как при каждой новой инициализации программы память относительно которой ведётся начало работы будет разной (новый запуск = ОС выделяет новый кусок памяти с другим адресом в рамках которого работает программа)

# Работа с указателями

Это конечно всё прекрасно, но как работать с `0xc00001206d` - ведь это всего лишь unit указывающий на память?

Для того чтобы взаимодействовать с данными находящимися "под" указателем используется механизм разъименовывания. Для того чтобы работать не с самим указателем, а с данными которые лежат под ним нужно использовать символ `*`

Например вот такой код

```go
package main

func main() {
	a := int8(6)
	b := &a 
  
	fmt.Println(a)
	*b = 5
	fmt.Println(a)
	fmt.Println(*b)
}
```

Даст следующий вывод

```
| 6
| 5
| 5
```

В данном случае происходит следующее:

Команда `*b = 5` - берёт адрес `0xc00001206d` и кладёт в него новое значение - пятёрку
Было
```
0000_0000110_00000101_...
    ^ а=6   ^ b=5     
```

Стало
```
	изменение
	|       |  	  
0000_0000101_00000101_...
    ^ а=5   ^ b=5     
```

Т.е символ `*` используемый как действие даёт нам возможность работать с данными указанного типа по указанному адресу.
# Nil pointer reference

В название подглавы выведена страшная ошибка работы с несуществующим куском памяти
Как и все типы, указатель имеет значение по умолчанию. Таким образом если определить переменную следующим образом:

```go
package main

func main() {
	var b *int
}
```

Переменная "b" примет (вернее сказать **не примет**) значение nil.

В первом примере значение для "b" мы получали в ходе взятия указателя на переменную "a". Здесь мы лишь определяем что в коде будет использоваться переменная с типом *указатель на целое число*, но сама по себе эта переменная никуда не указывает

Значит при попытке обратить в память, через переменную, которая никуда не указывает - мы получим критическую ошибку (панику), которая приведёт к падению всей программы

Всё просто!

# Указатель на практике

Рассмотрим две реализации функции add:

## Add (v1)
```go
package main

import "fmt"

func main() {
	var a int
	add(a)
	fmt.Println(a)
}

func add(a int) {
	a += 1
}
```

Можете сказать, что выведется?

Стоит вспомнить как работает стэк - при возвращении из функции, значение копируется из кадра вызванной функции в кадр вызывающей. В отношении передачи параметра действует тот же принцып!

При передачи чего либо в функцию это "что либо" копируется и только потом передаётся. Даже указатель, который является всего навсего числом копируется прежде чем будет передан в функцию.

Раз мы работаем с копией, то ответ очевиден - при вызове функции "add" переменна "а" будет скопирована, передана в функцию, там значение копии будет увеличено на 1, при этом основная переменная в функции main останется неизменной.

Исправим ситуацию с использованием приобретённых знаний:
## Add v2

Изменим сигнатуру функции - теперь она принимает указатель на целое число

```go
package main

import "fmt"

func main() {
	var a int
	add(&a)
	fmt.Println(a)
}

func add(aPtr *int) {
	*aPtr += 1
}
```

Стэк будет выглядеть следующим образом до прибавления единицы

```
main.add   | 00000101 | // aPtr = 5          
-----------------------
main.main: | 00000000 | // <- пускай это переменная "a" и её адрес 5
```

И вот так после

```
main.add   | 00000101 | // aPtr = 5          
-----------------------
main.main: | 00000001 |
```

Таким образом, мы смогли из памяти нижележащей (add) функции добраться до памяти вышестоящей функции (main)

# Бег указателя

Теперь попробуем провернуть действие в обратную сторону, не передавать указатель куда-то, а получать его из функции и делать с ним что-то

```go
package main

import "fmt"

func main() {
	a := getIntPtr(&a)
	*a += 1
	fmt.Println(*a)
}

func getIntPtr() *int {
	a := 0
	return &a
}
```

В рамках функции `getIntPtr` определяем переменную `a` и возвращаем указатель на неё в функцию `main`

Стэк до выхода из  `getIntPtr` выглядит вот так :

```
main.getIntPtr | 00000000 | // a          
	           | 00001010 | // указатель на a (пускай будет равен 10)
-----------------------
main.main:     | 00000000 | // память под указатель на целое число
```

После выхода из `getIntPtr` стэк **должен** принять следующий вид:

```
main.main:     | 00001010 | // память под указатель на целое число
```

Однако куда теперь указывает переменная полученная в результате выполнения функции? Ведь после выхода из функции кадр помечается как удалённый и может быть переиспользован другими функциями!

Таким образом работа с указателями работает, например, в С. Однако Go понимает, что переменная `а` определённая в `getIntPtr` используется за пределами функции (т.к мы возвращаем указатель на неё), поэтому среда выполнения алоцирует переменную не на стэке, а в куче и честнее будет показать стэк следующим образом

До выхода из функции:
```
main.getIntPtr | 00000000 | // a          
	           | 00001010 | // указатель на "a" (пускай будет равен 10)
-----------------------
main.main:     | 00000000 | // память под указатель на целое число
```

После выхода из функции:

```
main.main:     | 10000110 | // указатель на кучу
```

Куча в это время выглядит примерно так
```
...0000_00000000_0000....
       ^ 134 позиция (или 10000110 в двоичной системе)
```

Так как куча определяется далеко за пределами стэка можно не беспокоится, что данные в ней будут удалены при выходе из функции, однако стоит помнить, что за такое удобство (огромное удобство) мы платим дополнительными алокациями, а значит системным вызовом