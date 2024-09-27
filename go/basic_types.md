[[../index.md|<--Go Back]]

# = Go Basic Types =

* __int__ - целое положительное или отрицательное число (32 или 64 бита, зависит от платформы). Также есть типы определенного размера __int8__, __int16__, __int32__, __int64__
    - 20, -20, 0x14, 0o24, 0b0010100
* __uint__ - положительное целое число (32 или 64 бита, зависит от платформы). Также есть типы определенного размера __uint8__, __uint16__, __uint32__, __uint64__
    - нет литералов, все литеральные целые числа обрабатываются как __int__
* __byte__ - псевдоним __uint8__
    - нет литералов, выражаются как целочисленные литералы __101__ или литералы выполнения __'e'__
* __float32__, __float64__ - числа с дробью
    - 20.2, -20.2, 1.2е10, 1.2е-10
* __complex64__, __complex128__ - комплексные числа
* __bool__ - __true__/__false__
    - true, false
* __string__ - последовательность символов
    - "Hello", двойные кавычки - работают слеши, обратная кавычка - escape не интерпретируется
* __rune__ - одна кодовая точка __unicode__, псевдоним для __uint32__
    - 'A', '\n', '\u00A5', '¥' - __символы должны быть в одинарных кавычках__

# = Sensetive shit =	
- Не нужно указывать тип при использовании буквального значения, копмпилятор выведет тип на основе способа выражения значения:
    `fmt.Ptintln("Hello, Go!")`, компилятор поймет, что `"Hello, Go!"` это строка