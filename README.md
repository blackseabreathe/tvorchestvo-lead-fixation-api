# Творчество, API фиксации лидов
Документация описывает возможные ответы от API при попытке зарегистрировать лид. Ответы охватывают случаи ошибок валидации, проверку уникальности номера телефона и статус фиксации лида.

## Общая структура ответа
```
{
  "success": boolean
  "error": string                                             // При ошибке будет содержаться текст в данном параметре
  "unique": boolean                                           // Индикатор уникальности
  "unique_until": null|string                                 // Дата фиксации лида, возвращается строка вида YYYY-MM-DD, если в лиде заполнено поле или null в противном случае
}
```

## 🟡 Токен должен быть передан в заголовках запроса в параметре `x-token`

Запрос должен быть отправлен методом `POST`. 

## Формат тела запроса
Валидный JSON


### ❌ Ошибки в запросе (HTTP_CODE: 400, 401, 405)

#### 1. Отправлен запрос методом, отличным от `POST`
```
{
  "success": false,
  "error": "Wrong request method",
  "unique": false,
  "unique_until" => null
}
```

#### 2. Не передан токен
```
{
  "success": false,
  "error": "Token is missing",
  "unique": false,
  "unique_until" => null
}
```

#### 3. Некорректный токен
```
{
  "success": false,
  "error": "Token required",
  "unique": false,
  "unique_until" => null
}
```

#### 4. Тело запроса пустое
```
{
  "success": false,
  "error": "No data was transffered",
  "unique": false,
  "unique_until" => null
}
```




### ❌ Ошибки валидации (400)


#### 1. Отсутствует массив client
```
{
  "success": false,
  "error": "`client` is required",
  "unique": false,
  "unique_until" => null
}
```

#### 2. Отсутствует имя клиента
```
{
  "success": false,
  "error": "`client name` must be at least 5 characters",
  "unique": false,
  "unique_until" => null
}
```

#### 3. Отсутствует телефон клиента или передано менее 11 цифр:
```
{
  "success": false,
  "error": "`client phone` must be 11 characters",
  "unique": false,
  "unique_until" => null
}
```

#### 4. Отсутствует массив agent:
```
{
  "success": false,
  "error": "`agent` is required",
  "unique": false,
  "unique_until" => null
}
```

#### 5. Отсутствует имя агента:
```
{
  "success": false,
  "error": "`agent name` must be at least 5 characters",
  "unique": false,
  "unique_until" => null
}
```

#### 6. Отсутствует телефон агента или передано менее 11 цифр:
```
{
  "success": false,
  "error": "`agent phone` must be 11 characters",
  "unique": false,
  "unique_until" => null
}
```



### ⚠️ Статусы фиксации (HTTP_CODE: 200)

#### 1. Телефон не уникальный:
```
{
  "success": true,
  "error": "",
  "unique": false,
  "unique_until" => null|string
}
```

#### 2. Телефон уникальный:
```
{
  "success": true,
  "error": "",
  "unique": true,
  "unique_until" => string
}
```



# Curl пример запроса
```diff
- Внимание! Все параметры в запросе обязательны
```

```php
$endpoint = 'xxx/api/lead-fixation/create';

$params = [
    "agent" => [
        "name" => "Джеймс Бонд 007",  // мин 5 символов
        "phone" => "+7 (900) 000-11-44" // любой формат номера телефона, но состоящий из 11 цифр
    ],
    "client" => [
        "phone" => "7-900-0001133", // любой формат номера телефона, но состоящий из 11 цифр
        "name" => "Иванов Иван Иванович", // мин 5 символов
    ]
];

// Заголовки
$headers = [
    'Accept: application/json',
    'Content-Type: application/json',
    'x-token: xxxxxxxxxxxxxxxx'
];


$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_POST, 1);
curl_setopt($curl, CURLOPT_POSTFIELDS, json_encode($params));
curl_setopt($curl, CURLOPT_URL, $endpoint);
curl_setopt($curl, CURLOPT_HEADER, 0);
curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
curl_setopt($curl, CURLOPT_CONNECTTIMEOUT, 10); // рекомендуемые таймауты 10
curl_setopt($curl, CURLOPT_TIMEOUT, 10); // рекомендуемые таймауты 10

$result = json_decode(curl_exec($curl), true);
$http_code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
$curl_errno = curl_errno($curl);
curl_close($curl);

echo "http code: $http_code<br><br><br>";

echo '<pre>';
print_r($result);
```
