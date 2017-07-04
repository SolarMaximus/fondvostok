# OAuth2.0 Privatization 

## GET Authorization Request

Клиент формирует GET-запрос к странице Login.aspx, добавляя следующие параметры:
1. **response_type** - должен быть передан "code".
2. **client_id** - идентификатор клиента. Выдается клиенту, чтобы идентифицировать себя на сервере авторизации. 
3. **redirect_uri** - URL, на который будет перенаправлен пользователь.
4. **state** - идентификатор сессии клиента. Является опциональным. 

Пример GET-запроса:
http://domain/Login.aspx?client_id=1234abc&response_type=code&redirect_uri=http://yandex.ru

## Authorization Response

Сервер перенаправляет на redirect_uri, добавляя к нему следующие параметры:
1. **code** - код авторизации, генерируется на сервере. Время жизни кода - 300 секунд. Удаляется после первого использования. Код привязан к client_id и redirect_uri. 
2. **state** - если state был передан в AuthorizationRequest. Возвращается то же самое значение, что было передано ранее.

Если при получении кода произошла ошибка, то к redirect_uri добавляются параметры:
1. **error** - код ошибки. 
2. **error_description** - словесное описание ошибки.
3. **state** - если state был передан в AuthorizationRequest. Возвращается то же самое значение, что было передано ранее.

Код ответа - 302. 

## POST Access Token Request

Клиент формирует POST-запрос к Login.aspx/GetAccessToken, передавая следующие параметры:
1. **client_id** - идентификатор клиента. 
2. **client_secret** - пароль клиента.
3. **redirect_uri** - URL, на который сервер вернул Authorization Response. 
4. **grant_type** - должен быть передан "authorization_code".
5. **code** - код авторизации, полученный на предыдущем шаге. 


Пример запроса:

```javascript
{
  client_id: "1234abc",
  client_secret: "2345bcd",
  code: "veryuniquevalue",
  grant_type: "authorization_code",
  redirect_uri: "http://yandex.ru"
}
```

## Access Token Response

Если все параметры запроса валидные и клиент аутентифицирован, то ответ содержит следующие параметры:
1. **access_token** - ключ для авторизации. 
2. **token_type** - передается "bearer".
3. **expires_in** - время жизни Access_token в секундах (на данный момент 3600). 
4. **refresh_token** - токен, через который происходит обновление access_token. Время жизни - 10 недель.

Код ответа - 200. 

Если при выполнении запроса произошла ошибка, то ответ содержит следующие параметры:
1. **error** - код ошибки. 
2. **error_description** - словесное описание ошибки.

Код ответа - 400 или 500. 

Пример ответа:

```javascript
{
  error: "invalid_client",
  error_description: "Client not found",
}
```

## POST User Profile Request

Клиент формирует POST-запрос к Login.aspx/UserProfile, передавая следующие параметры:
1. **access_token** - ключ для авторизации. 

Пример запроса:

```javascript
{
  "access_token": "pSixiapzz06SgFWd5cJMEcs7wcRNyRTY1ZzCvnUMjrDv_WsSG4mmfRLOeW_mToNcJSAIiRbULJVGP8XsB841iA"
}
```

## User Profile Response

Если запрос успешно обработан, то ответ содержит следующие параметры:
1. **username** - передается уникальное имя пользователя. 
2. **email** - передается email пользователя.

Код ответа - 200. 

Если при выполнении запроса произошла ошибка, то ответ содержит следующие параметры:
1. **error** - код ошибки. 
2. **error_description** - словесное описание ошибки.

Код ответа - 404. 

Пример ответа:

```javascript
{
  error: "user_not_found",
  error_description: "User not found by access token"
}
```

## POST Refresh Token Request

Клиент формирует POST-запрос к Login.aspx/RefreshToken, передавая следующие параметры:
1. **client_id** - идентификатор клиента. 
2. **client_secret** - пароль клиента.
3. **refresh_token** - токен, через который происходит обновление access_token.
4. **grant_type** - должен быть передан "refresh_token".

Пример ответа:

```javascript
{
  client_id: "1234abc",
  client_secret: "2345bcd",
  refresh_token: "YourRefreshToken",
  grant_type: "refresh_token",
}
```

## Refresh Token Response

Формат аналогичный AccessTokenResponse.
Если все параметры валидные, то передаются новые access_token и refresh_token. 
После обновления старые токены на стороне пользователя должны быть удалены/заблокированы. 
