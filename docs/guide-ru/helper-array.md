ArrayHelper
===========
Вдобавок к [богатому набору функций](https://www.php.net/manual/ru/book.array.php)  для работы с массивами, которые есть в самом PHP, хелпер Yii Array предоставляет свои статические функции - возможно они могут быть вам полезны.


## Получение значений <span id="getting-values"></span>

Извлечение значений из массива, объекта или структуры состоящей из них обоих с помощью стандартных средств PHP является довольно скучным занятием. Сначала вам нужно проверить, есть ли соответствующий ключ с помощью `isset`, и если есть – получить, если нет – подставить значение по умолчанию.

```php
class User
{
    public $name = 'Alex';
}

$array = [
    'foo' => [
        'bar' => new User(),
    ]
];

$value = isset($array['foo']['bar']->name) ? $array['foo']['bar']->name : null;
```

Yii предлагает очень удобный метод для таких случаев:

```php
$value = ArrayHelper::getValue($array, 'foo.bar.name');
```

Первый аргумент – массив или объект из которого мы извлекаем значение. Второй аргумент определяет как будут извлекаться данные и может выглядеть как один из таких вариантов:
- Имя ключа массива или свойства объекта, значение которого нужно вернуть
- Путь к нужному значению, разделенный точками, как в примере выше
- Callback-функция, возвращающая значение

Callback-функция должна выглядеть примерно так:

```php
$fullName = ArrayHelper::getValue($user, function ($user, $defaultValue) {
    return $user->firstName . ' ' . $user->lastName;
});
```

Третий необязательный аргумент определяет значение по умолчанию. Если не установлен – равен `null`. Используется так:

```php
$username = ArrayHelper::getValue($comment, 'user.username', 'Unknown');
```


## Запись значений <span id="setting-values"></span>

```php
$array = [
    'key' => [
        'in' => ['k' => 'value']
    ]
];

ArrayHelper::setValue($array, 'key.in', ['arr' => 'val']);
// путь для записи значения в `$array` можно указать как массив
ArrayHelper::setValue($array, ['key', 'in'], ['arr' => 'val']);
```

В результате исходное значение `$array['key']['in']` будет перезаписано новым

```php
[
    'key' => [
        'in' => ['arr' => 'val']
    ]
]
```

Если путь содержит несуществующий ключ, то он будет создан

```php
// Если `$array['key']['in']['arr0']` не пустой, то значение будет добавлено в массив
ArrayHelper::setValue($array, 'key.in.arr0.arr1', 'val');

// если необходимо полностью переопределить значение `$array['key']['in']['arr0']`
ArrayHelper::setValue($array, 'key.in.arr0', ['arr1' => 'val']);
```

Результатом будет следующим:

```php
[
    'key' => [
        'in' => [
            'k' => 'value',
            'arr0' => ['arr1' => 'val']
        ]
    ]
]
```


## Изъять значение из массива <span id="removing-values"></span>

Если вы хотите получить значение и тут же удалить его из массива, вы можете использовать метод `remove`

```php
$array = ['type' => 'A', 'options' => [1, 2]];
$type = ArrayHelper::remove($array, 'type');
```

После выполнения этого кода переменная `$array` будет содержать `['options' => [1, 2]]`, а в переменной `$type` будет значение `А`. В отличие от метода `getValue`, метод `remove` поддерживает только простое имя ключа.


## Проверка наличия ключей <span id="checking-existence-of-keys"></span>

`ArrayHelper::keyExists` работает так же, как и стандартный [array_key_exists](https://www.php.net/manual/ru/function.array-key-exists.php),
но также может проверять ключи без учёта регистра:

```php
$data1 = [
    'userName' => 'Alex',
];

$data2 = [
    'username' => 'Carsten',
];

if (!ArrayHelper::keyExists('username', $data1, false) || !ArrayHelper::keyExists('username', $data2, false)) {
    echo "Пожалуйста, укажите имя пользователя.";
}
```

## Извлечение столбцов <span id="retrieving-columns"></span>

Часто нужно извлечь столбец значений из многомерного массива или объекта. Например, список ID.

```php
$array = [
    ['id' => '123', 'data' => 'abc'],
    ['id' => '345', 'data' => 'def'],
];
$ids = ArrayHelper::getColumn($array, 'id');
```

Результатом будет `['123', '345']`.

Если нужны какие-то дополнительные трансформации или способ получения значения специфический, вторым аргументом может быть анонимная функция:

```php
$result = ArrayHelper::getColumn($array, function ($element) {
    return $element['id'];
});
```

## Переиндексация массивов <span id="reindexing-arrays"></span>


Чтобы проиндексировать массив в соответствии с определенным ключом, используется метод `index`. Входящий массив должен
быть многомерным или массивом объектов. Ключом может быть имя ключа вложенного массива, имя свойства объекта или
анонимная функция, которая будет возвращать значение ключа по переданному элементу индексируемого массива (то есть по
вложенному массиву или объекту).

Если значение ключа равно `null`, то соответствующий элемент массива будет опущен и не попадет в результат.

```php
$array = [
    ['id' => '123', 'data' => 'abc'],
    ['id' => '345', 'data' => 'def'],
];
$result = ArrayHelper::index($array, 'id');
// результат будет таким:
// [
//     '123' => ['id' => '123', 'data' => 'abc'],
//     '345' => ['id' => '345', 'data' => 'def'],
// ]

// использование анонимной функции
$result = ArrayHelper::index($array, function ($element) {
    return $element['id'];
});
```



## Получение пар ключ-значение <span id="building-maps"></span>

Для получения пар ключ-значение из многомерного массива или из массива объектов вы можете использовать метод `map`.

Параметры `$from` и `$to` определяют имена ключей или свойств, которые будут использованы в map. Также третьим необязательным параметром вы можете задать правила группировки.

```php
$array = [
    ['id' => '123', 'name' => 'aaa', 'class' => 'x'],
    ['id' => '124', 'name' => 'bbb', 'class' => 'x'],
    ['id' => '345', 'name' => 'ccc', 'class' => 'y'],
);

$result = ArrayHelper::map($array, 'id', 'name');
// результат будет таким:
// [
//     '123' => 'aaa',
//     '124' => 'bbb',
//     '345' => 'ccc',
// ]

$result = ArrayHelper::map($array, 'id', 'name', 'class');
// результат будет таким:
// [
//     'x' => [
//         '123' => 'aaa',
//         '124' => 'bbb',
//     ],
//     'y' => [
//         '345' => 'ccc',
//     ],
// ]
```


## Многомерная сортировка <span id="multidimensional-sorting"></span>

Метод `multisort` помогает сортировать массивы объектов или вложенные массивы по одному или нескольким ключам. Например:

```php
$data = [
    ['age' => 30, 'name' => 'Alexander'],
    ['age' => 30, 'name' => 'Brian'],
    ['age' => 19, 'name' => 'Barney'],
];
ArrayHelper::multisort($data, ['age', 'name'], [SORT_ASC, SORT_DESC]);
```

После сортировки мы получим:

```php
[
    ['age' => 19, 'name' => 'Barney'],
    ['age' => 30, 'name' => 'Brian'],
    ['age' => 30, 'name' => 'Alexander'],
];
```

Второй аргумент, определяющий ключи для сортировки может быть строкой, если это один ключ, массивом, если используются несколько ключей или анонимной функцией, как в примере ниже:

```php
ArrayHelper::multisort($data, function($item) {
    return isset($item['age']) ? ['age', 'name'] : 'name';
});
```
Третий аргумент определяет способ сортировки – от большего к меньшему или от меньшего к большему. В случае, если мы сортируем по одному ключу, передаем  `SORT_ASC`  или  `SORT_DESC`. Если сортировка осуществляется по нескольким ключам, вы можете назначить направление сортировки для каждого из них с помощью массива.

Последний аргумент – это флаг, который используется в стандартной функции PHP `sort()`. Посмотреть его возможные значения можно [тут](https://www.php.net/manual/ru/function.sort.php).


## Определение типа массива <span id="detecting-array-types"></span>

Удобный способ для определения, является массив индексным или ассоциативным. Вот пример:

```php
// ключи не определены
$indexed = ['Qiang', 'Paul'];
echo ArrayHelper::isIndexed($indexed);

// все ключи - строки
$associative = ['framework' => 'Yii', 'version' => '2.0'];
echo ArrayHelper::isAssociative($associative);
```


## HTML-кодирование и HTML-декодирование значений <span id="html-encoding-values"></span>


Для того, чтобы закодировать или раскодировать специальные символы в массиве строк в HTML-сущности, вы можете пользоваться методами ниже:

```php
$encoded = ArrayHelper::htmlEncode($data);
$decoded = ArrayHelper::htmlDecode($data);
```

По умолчанию кодируются только значения. Если установить второй параметр в `false`, то ключи массива будут так же кодированы. Кодирование использует кодировку приложения, которая может быть изменена с помощью третьего аргумента.

## Слияние массивов <span id="merging-arrays"></span>

Слияние двух или больше массивов в один рекурсивно.
Если каждый массив имеет одинаковый ключ, последний будет перезаписывать предыдущий (в отличие от функции array_merge_recursive).
Рекурсивное слияние проводится когда все массивы имеют элемент одного и того же типа с одним и тем же ключом. Для элементов, ключом которого является значение типа integer, элементы из последнего будут добавлены к предыдущим массивам. Вы можете добавлять дополнительные массивы для слияния третьим, четвертым, пятым (и так далее) параметром.

```php
ArrayHelper::merge($a, $b);
```

## Получение массива из объекта <span id="converting-objects-to-arrays"></span>

Часто нужно конвертировать объект в массив. Наиболее распространенный случай – конвертация модели Active Record в массив.

```php
$posts = Post::find()->limit(10)->all();
$data = ArrayHelper::toArray($posts, [
    'app\models\Post' => [
        'id',
        'title',
        // ключ в результирующем массиве => имя свойства
        'createTime' => 'created_at',
        // ключ в результирующем массиве => анонимная функция
        'length' => function ($post) {
            return strlen($post->content);
        },
    ],
]);
```

Первый аргумент содержит данные, которые вы хотите конвертировать. В нашем случае это модель Active Record `Post`.

Второй аргумент служит для управления процессом конвертации и может быть трех видов:

- просто имя поля
- пара ключ-значение, где ключ определяет ключ в результирующем массиве, а значение – название поля в модели, откуда берется значение.
- пара ключ-значение, где в качестве значения передается callback-функция, которая возвращает значение.

Результат конвертации будет таким:

```php
[
    'id' => 123,
    'title' => 'test',
    'createTime' => '2013-01-01 12:00AM',
    'length' => 301,
]
```

Вы можете определить способ конвертации из объекта в массив по-умолчанию реализовав интерфейс
[[yii\base\Arrayable|Arrayable]] в этом классе


## Проверка на присутствие в массиве <span id="testing-arrays"></span>

Часто необходимо проверить, содержится ли элемент в массиве, или является ли массив подмножеством другого массива.
К сожалению, PHP-функция `in_array()` не поддерживает подмножества объектов, реализующих интерфейс `\Traversable`.

Для таких случаев [[yii\helpers\ArrayHelper]] предоставляет [[yii\helpers\ArrayHelper::isIn()|isIn()]] и
[[yii\helpers\ArrayHelper::isSubset()|isSubset()]]. Методы принимают такие же параметры, что и
[in_array()](https://www.php.net/manual/ru/function.in-array.php).

```php
// true
ArrayHelper::isIn('a', ['a']);
// true
ArrayHelper::isIn('a', new(ArrayObject['a']));

// true
ArrayHelper::isSubset(new(ArrayObject['a', 'c']), new(ArrayObject['a', 'b', 'c'])

```

## Преобразование многомерных массивов <span id="flattening-arrays"></span>

Метод `ArrayHelper::flatten()` позволяет преобразовать многомерный массив в одномерный, объединяя ключи.

### Основное использование

Чтобы преобразовать вложенный массив, просто передайте массив в метод `flatten()`:

```php
$array = [
    'a' => [
        'b' => [
            'c' => 1,
            'd' => 2,
        ],
        'e' => 3,
    ],
    'f' => 4,
];

$flattenedArray = ArrayHelper::flatten($array);
// Результат:
// [
//     'a.b.c' => 1,
//     'a.b.d' => 2,
//     'a.e' => 3,
//     'f' => 4,
// ]
```

### Пользовательский разделитель

Вы можете указать пользовательский (т.е. отличный от значения по умолчанию: `.`) разделитель для объединения ключей:

```php
$array = [
    'a' => [
        'b' => [
            'c' => 1,
            'd' => 2,
        ],
        'e' => 3,
    ],
    'f' => 4,
];

$flattenedArray = ArrayHelper::flatten($array, '_');
// Результат:
// [
//     'a_b_c' => 1,
//     'a_b_d' => 2,
//     'a_e' => 3,
//     'f' => 4,
// ]
```

### Обработка специальных символов в ключах

Метод `flatten()` может обрабатывать ключи со специальными символами:

```php
$array = [
    'a.b' => [
        'c.d' => 1,
    ],
    'e.f' => 2,
];

$flattenedArray = ArrayHelper::flatten($array);
// Результат:
// [
//     'a.b.c.d' => 1,
//     'e.f' => 2,
// ]
```

### Смешанные типы данных

Метод `flatten()` работает с массивами, содержащими различные типы данных:

```php
$array = [
    'a' => [
        'b' => 'string',
        'c' => 123,
        'd' => true,
        'e' => null,
    ],
    'f' => [1, 2, 3],
];

$flattenedArray = ArrayHelper::flatten($array);
// Результат:
// [
//     'a.b' => 'string',
//     'a.c' => 123,
//     'a.d' => true,
//     'a.e' => null,
//     'f.0' => 1,
//     'f.1' => 2,
//     'f.2' => 3,
// ]
```

### Краевые случаи

Метод `flatten()` обрабатывает различные краевые случаи, такие как пустые массивы и значения, не являющиеся массивами:

```php
// Пустой массив
$array = [];
$flattenedArray = ArrayHelper::flatten($array);
// Результат: []

// Значение, не являющееся массивом
$array = 'string';
$flattenedArray = ArrayHelper::flatten($array);
// Результат:
// yii\base\InvalidArgumentException: Argument $array must be an array or implement Traversable
```

### Коллизии ключей

Когда ключи совпадают, метод `flatten()` перезапишет предыдущее значение:

```php
$array = [
    'a' => [
        'b' => 1,
    ],
    'a.b' => 2,
];

$flattenedArray = ArrayHelper::flatten($array);
// Результат: ['a.b' => 2]
```
