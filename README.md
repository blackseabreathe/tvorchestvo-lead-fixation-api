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

Во всех ответах HTTP статус идентичен параметру `error`, кроме успешной фиксации. Например, не отправили номер телефона агента, в ответе получили `{... "error": 400, ...}`, значит и HTTP статус тоже 400.

При успешной фиксации 
```diff 
+ HTTP статус - 200
```
параметр `error` не возвращается.

В параметре `message` возвращается полное описание ошибки


# Curl пример запроса
```diff
- Внимание! Есть обязательные параметры в запросе
```

```php
$endpoint = 'xxx/local/samoliot-app/lead-fixation/fixation/v2/';

$params = [
    "phone" => "+7 (912) 000-00-11", // Номер телефона клиента, любой формат номера телефона, но состоящий из 11 цифр (обязательный)
    "surname" => "Безфамильный", // Фамилия клиента (обязательный)
    "name" => "Безимян", // Имя клиента (обязательный)
    "middleName" => "Безотчествович", // Отчество клиента
    "agency" => "Ми-6", // Название агентства (обязательный)
    "agent" => [
        "name" => "Агент Джеймс Бонд 007", // Имя агента (обязательный)
        "phone" => "7 900 232 52-23", // Телефон агента, любой формат номера телефона, но состоящий из 11 цифр (обязательный)
    ],
];

// Заголовки
$headers = [
    'Accept: application/json', // формат JSON
    'Content-Type: application/json', // формат JSON
    'X-AGENT-TOKEN: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx'
];


$curl = curl_init();
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_URL, $endpoint.'?'.http_build_query($params));
curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);

$result = json_decode(curl_exec($curl), true);
if (curl_errno($curl)) {
    echo 'Ошибка cURL: ' . curl_error($curl);
}

$http_code = curl_getinfo($curl, CURLINFO_HTTP_CODE); // получим HTTP статус
curl_close($curl);

echo "http code: $http_code<br><br><br>";

echo '<pre>';
print_r($result);
```
