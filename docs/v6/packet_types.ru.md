# Типы пакетов

* Версия 6 не совместима с ранними версиями протокола.
* Узлы с версией 6 будут отвечат узлам старшей версии.
* Строки в пакетах имеют кодировку UTF-8.
* Литера `E` означает, что данные зашифрованы.

Информация о поддерживаемых алгоритмах шифрования и подписи находится в разделе [Криптография](cryptography.md)

## 1. Пакеты данных

**Все** `Пакеты данных` начинаются с:

- `Типа пакета` (1 байт);
- `Версии протокола` (1 байт).

**Все** `Пакеты данных` **всегда** пересылаются обёрнутыми в `Коммутационные Пакеты` (2.).

### 1.1 Почтовый пакет (шифрованый)

Письмо или фрагмент письма для одного получателя.  
Часть пакета зашифрована ключом получателя.

| Поле    | Размер     | Описание                                                            |
|---------|------------|---------------------------------------------------------------------|
| `TYPE`  | 1 байт     | Значение = `'E'`                                                    |
| `VER`   | 1 байт     | Версия протокола                                                    |
| `KEY`   | 32 байт    | DHT-ключ пакета, SHA-256 хэш полей `LEN+DATA`                       |
| `TIM`   | 8 байт     | Дата, когда пакет был сохранён на узле (int64) `[VER 6]`            |
| `DV`    | 32 байт    | Хэш Подтверждения Удаления, SHA-256 хеш поля `DA`. Также используеся как контрольная сумма для валидации расшифрованного пакета |
| `ALG`   | 1 байт     | Идентификатор алгоритма, используемого для шифрования.              |
| `LEN`   | 2 байт     | Длина поля `DATA`                                                   |
| `DATA`  | байт[] `E` | Данные для расшифровки и `Почтовый пакет (расшифрованый)`           |

ToDo: add info about alg-specific prefixes

### 1.2 Почтовый пакет (расшифрованый)

Формат хранения `Неполных Писем` (письма, для которых не хватает частей для компоновки целого письма).

| Поле    | Размер     | Описание                                            |
|---------|------------|-----------------------------------------------------|
| `TYPE`  | 1 байт     | Значение = `'U'`                                    |
| `VER`   | 1 байт     | Версия протокола                                    |
| `MSID`  | 32 байт    | Идентификатов письма в двоичном формате             |
| `DA`    | 32 байт    | Ключ Подтверждения Удаления, случайно сгенерирован. |
| `FRID`  | 2 байт     | Порядковый номер части письма (0..`NFR`-1)          |
| `NFR`   | 2 байт     | Количество частей письма.                           |
| `MLEN`  | 2 байт     | Длина полей `CALG+MSG`                              |
| `CALG`  | 1 байт     | Алгоритм сжатия (см. ниже)                          |
| `MSG`   | байт[]     | Содержание письма (`MLEN` байт)                     |

#### Алгоритмы сжатия

В зависимости от реализации поддерживаются различные алгоритмы сжатия.

| Значение | Описание     | i2p.i2p-bote| pboted |
|----------|--------------|-------------|--------|
| `0`      | Uncompressed | да          | да     |
| `1`      | LZMA         | да          | да     |
| `2`      | ZLIB         | нет         | да     |
| `3`      | BZIP2        | нет         | нет    |
| `4`      | ZSTD         | нет         | нет    |
| `5`      | LZ4          | нет         | нет    |
| `6`      | Snappy       | нет         | нет    |

### 1.3 Индексный пакет

Содержит DHT-ключи для одного и более `Почтовых пакетов`, `Хэш Подтверждения Удаления` и временную метку UNIX для каждого элемента.

| Поле    | Размер    | Описание                                                                                           |
|---------|-----------|----------------------------------------------------------------------------------------------------|
| `TYPE`  | 1 байт    | Значение = `'I'`                                                                                   |
| `VER`   | 1 байт    | Версия протокола                                                                                   |
| `DH`    | 32 байт   | SHA-256 хэш `Почтового Назначения`                                                                 |
| `NP`    | 4 байт    | Число записей                                                                                      |
| `KEY1`  | 32 байт   | DHT-ключ первого `Почтового пакета`                                                                |
| `DV1`   | 32 байт   | `Хэш Подтверждения Удаления` для `KEY1` (SHA-256 хэш `Авторизации Удаления` их `Почтового пакета`) |
| `TIM1`  | 8 байт    | Время, в которое `KEY1` был добавлен на узел. (int64)  `[VER 6]`                                   |
| `KEY2`  | 32 байт   | DHT-ключ второго `Почтового пакета`                                                                |
| `DV2`   | 32 байт   | `Хэш Подтверждения Удаления` для `KEY2` (SHA-256 хэш `Авторизации Удаления` их `Почтового пакета`) |
| `TIM2`  | 8 байт    | Время, в которое `KEY2` был добавлен на узел. (int64)  `[VER 6]`                                   |
| ...     | ...       | ...                                                                                                |
| `KEYn`  | 32 байт   | DHT-ключ n-ного `Почтового пакета`                                                                 |
| `DVn`   | 32 байт   | `Хэш Подтверждения Удаления` для `KEYn` (SHA-256 хэш `Авторизации Удаления` их `Почтового пакета`) |
| `TIMn`  | 8 байт    | Время, в которое `KEYn` был добавлен на узел. (int64)  `[VER 6]`                                   |

### 1.4 Информационный пакет удаления

Содержит информацию об удалённых из DHT элементах, которые могут быть `Почтовыми пакетами` или `Индексными пакетами`.

Содержится в `Пакете ответа` на `Запрос удаления`.

| Поле    | Размер  | Описание                                                 |
|---------|---------|----------------------------------------------------------|
| `TYPE`  | 1 байт  | Значение = `'T'`                                         |
| `VER`   | 1 байт  | Версия протокола                                         |
| `NP`    | 4 байт  | Число записей                                            |
| `KEY1`  | 32 байт | Первый DHT-ключ                                          |
| `DA1`   | 32 байт | `Авторизация Удаления` для `KEY1`                        |
| `TIM1`  | 8 байт  | Время, в которое `KEY1` был добавлен. (int64)  `[VER 6]` |
| `KEY2`  | 32 байт | Второй DHT-ключ                                          |
| `DA2`   | 32 байт | `Авторизация Удаления` для `KEY2`                        |
| `TIM2`  | 8 байт  | Время, в которое `KEY2` был добавлен. (int64)  `[VER 6]` |
| ...     | ...     | ...                                                      |
| `KEYn`  | 32 байт | n-ный DHT-ключ                                           |
| `DAn`   | 32 байт | `Авторизация Удаления` для `KEYn`                        |
| `TIMn`  | 8 байт  | Время, в которое `KEYn` был добавлен. (int64)  `[VER 6]` |

### 1.5 Список пиров

Является ответом на `Запрос ближайших узлов` или на `Запрос списка пиров`.

| Поле    | Размер     | Описание                                    |
|---------|------------|---------------------------------------------|
| `TYPE`  | 1 byte     | Значение = `'L'`                            |
| `VER`   | 1 byte     | Версия протокола.                           |
| `NUMP`  | 2 bytes    | Число записей в пакете.                     |
| `IDN1`  | 384 bytes  | Стандартное `I2P назначение`.               |
| `TYPE1` | 1 byte     | Тип сертификата.                            |
| `LEN1`  | 2 byte     | Длина дополнительных данных в поле `DATA1`. |
| `DATA1` | byte[]     | Дополнительные данные.                      |
| `IDN2`  | 384 bytes  | Стандартное `I2P назначение`.               |
| `TYPE2` | 1 byte     | Тип сертификата.                            |
| `LEN2`  | 2 byte     | Длина дополнительных данных в поле `DATA2`. |
| `DATA2` | byte[]     | Дополнительные данные.                      |
| ...     | ...        | ...                                         |
| `IDNn`  | 384 bytes  | Стандартное `I2P назначение`.               |
| `TYPEn` | 1 byte     | Тип сертификата.                            |
| `LENn`  | 2 byte     | Длина дополнительных данных в поле `DATAn`. |
| `DATAn` | byte[]     | Дополнительные данные.                      |

### 1.6 Контакт

Преобразование имен и `Почтовых назначений`.

| Поле    | Размер     | Описание                                                         |
|---------|------------|------------------------------------------------------------------|
| `TYPE`  | 1 byte     | Значение = `'C'`                                                 |
| `VER`   | 1 byte     | Версия протокола.                                                |
| `KEY`   | 32 bytes   | DHT-ключ (SHA-256 хеш имени в кодировке UTF-8 в нижнем регистре) |
| `DLEN`  | 2 bytes    | Длина поля `DEST`.                                               |
| `DEST`  | byte[]     | `Почтовое назначение`, 8 bit (не Base64)                         |
| `SALT`  | 4 bytes    | A value such that scrypt(KEY|DEST|SALT) ends with a zero byte    |
| `PLEN`  | 2 bytes    | Длина поля `PIC`, максимум: 8192                                 |
| `PIC`   | byte[]     | Аватар в формате, совместимом с браузерами (JPG, PNG, GIF)       |
| `COMP`  | 1 byte     | Тип сжатия для `TEXT`                                            |
| `TLEN`  | 2 bytes    | Длина поля `TEXT`, максимум: 2048                                |
| `TEXT`  | byte[]     | Текст, заданый пользователем в кодировке UTF8                    |

## 2. Коммутационные пакеты

`Коммутационные пакеты` используются для общения `узлов` I2P/Bote между собой и для обмена информацией и данными.

Могут содержать внутри  `Пакеты данных`, пункт 3.1.

Все `Коммутационные пакеты` начинаются со:

- специального `Префикса` (4 байт);
- `Типа пакета` (1 байт);
- `Версии протокола` (1 байт);
- `Идентификатора сессии` (32 байт).

### 2.1 Запрос реле

Пакет, сообщающий получателю о необходимости связи с одноранговым узлом или одноранговыми узлами от имени отправителя.

Он содержит `I2P назначения`, время задержки и `Коммутационный пакет`

| Поле    | Размер     | Описание                                        |
|---------|------------|-------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`      |
| `TYPE`  | 1 byte     | Значение = `'R'`                                |
| `VER`   | 1 byte     | Версия протокола.                               |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов |
| `HLEN`  | 2 bytes    | Длина поля `HK`                                 |
| `HK`    | byte[]     | HashCash (`HLEN` байт)                          |
| `DLY`   | 4 bytes    | Задержка в секундах перед отправкой             |
| `NEXT`  | 384 bytes  | Пункт назначения для пересылки пакета           |
| `RET`   | Ret. Chain | Пустая или непустая цепочка возврата            |
| `DLEN`  | 2 bytes    | Длина поля `DATA`                               |
| `DATA`  | byte[] `E` | Зашифрованая с помощью ключа получателя. Может содержать `Запрос реле`, `Запрос на хранение` или `Запрос на удаление`. |
| `PAD`   | byte[]     | Опциональный отступ                             |

### 2.2 Запрос на возврат реле

Содержит `Цепочку возврата` и полезную нагрузку. Пункт назначения неизвестен для отправителя или любого другого хопа.

Используется для ответа на `Релейный пакет данных`.

| Поле    | Размер     | Описание                                                                |
|---------|------------|-------------------------------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`                              |
| `TYPE`  | 1 byte     | Значение = `'K'`                                                        |
| `VER`   | 1 byte     | Версия протокола.                                                       |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для взаимодействия между двумя узлами. Новый идентификатор на каждый хор. |
| `RET`   | Ret. Chain | Не пустая цепочка возврата.                                             |
| `DLEN`  | 2 bytes    | Длина поля `DATA`.                                                      |
| `DATA`  | byte[] `E` | Полезная нагрузка. Шифрована AES-256 на каждом хопе в цепочке возврата. |

### 2.3 Цепочка возврата

Не является пакетом сама по себе. Только как часть `Запроса реле` или `Запроса на возврат реле`.

| Поле    | Размер         | Описание                                                      |
|---------|----------------|---------------------------------------------------------------|
| `RL1`   | 2 bytes        | Длина поля `RET1`. Ноль означает, что цепь пуста.             |
| `RET1`  | `RL1` bytes    | Хоп 1 в цепочке возрата. Contains a zero byte, followed by the hop's `I2P destination` and an AES key to encrypt the payload with. |
| `RL2`   | 2 bytes `E`    | Length of `RET2`                                              |
| `RET2`  | `RL2` bytes `E`| Hop 2 in the return chain. Contains a zero byte, followed by the hop's `I2P destination` and an AES key to encrypt the payload with. This field is encrypted with the PK of hop 2. |
| `RL3`   | 2 bytes   `E`  | Length of `RET3`                                              |
| `RET3`  | `RL3` bytes `E`| Hop 3 in the return chain. Contains a zero byte, followed by the hop's `I2P destination` and an AES key to encrypt the payload with. This field is encrypted with the PKs of hops 3 and 2. |
| ...     | ...            | ...                                                           |
| `RLn`   | 2 bytes   `E`  | Length of `RETn`                                              |
| `RETn`  | `RLn` bytes `E`| Last hop in the return chain. Contains a zero byte, followed by the hop's `I2P destination` and an AES key to encrypt the payload with. This field is encrypted with the PKs of hops n...2. |
| `RLn+1` | 2 bytes   `E`  | Length of `RETn+1`                                            |
| `RETn+1`| `RLn+1` byt `E`| Number of hops (>0) followed by all AES256 keys (one per hop) |
| `RLn+2` | 2 bytes        | Value=0 to indicate this is the end of the return chain       |

### 2.4 Запрос на получение

Запрос к финальной точке на получение `Индексного пакета` или `Почтового пакета`.

В ответ ожидается `Пакет ответа` с корректным статусом.

| Поле    | Размер     | Описание                                               |
|---------|------------|--------------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`.            |
| `TYPE`  | 1 byte     | Значение = `'G'`.                                      |
| `VER`   | 1 byte     | Версия протокола.                                      |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов.       |
| `DTYP`  | 1 byte     | Тип данных для получения.                              |
| `KEY`   | 32 bytes   | DHT-ключ данных.                                       |
| `KPR`   | 384 bytes  | Email keypair of recipient                             |
| `RLEN`  | 2 bytes    | Length of the `RET` field                              |
| `RET`   | byte[]     | Relay Packet, contains the return chain (`RLEN` bytes) |

### 2.5 Пакет ответа

Ответ на `Запрос на получение`, `Запрос на доставку`, `Запрос ближайших узлов` или `Запрос пиров`.

| Поле    | Размер     | Описание                                             |
|---------|------------|------------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`.          |
| `TYPE`  | 1 byte     | Значение = `'N'`.                                    |
| `VER`   | 1 byte     | Версия протокола.                                    |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов.     |
| `STA`   | 1 byte     | Код состояния (смотри ниже).                         |
| `DLEN`  | 2 bytes    | Длина поля `DATA`. При значении 0 - пакет без данных.|
| `DATA`  | byte[]     | `Пакет данных`.                                      |

#### Коды состояния

| Значение | Описание                            |
|----------|-------------------------------------|
| `0`      | OK                                  |
| `1`      | Общая ошибка                        |
| `2`      | Данные не найдены                   |
| `3`      | Недействительный пакет              |
| `4`      | Недействительный HashCash           |
| `5`      | Предоставлено недостаточно HashCash |
| `6`      | Не осталось места на диске          |
| `7`      | Дублированные данные                |

### 2.6 Запрос списка одноранговых узлов

Запрос списка пиров-ретрансляторов высокой доступности.

`Пакет ответа` с корректным статусом ожидается в ответ.

| Поле    | Размер     | Описание                                         |
|---------|------------|--------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`.      |
| `TYPE`  | 1 byte     | Значение = `'A'`.                                |
| `VER`   | 1 byte     | Версия протокола.                                |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов. |

## 3 Коммутационные пакеты DHT

### 3.1 Запрос на извлечение

Запрос к узлу на получение данных для предоставленного типа данных и DHT-ключа к нему.

`Пакет ответа` с корректным статусом ожидается в ответ.

| Поле    | Размер     | Описание                                         |
|---------|------------|--------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`.      |
| `TYPE`  | 1 byte     | Значение = `'Q'`.                                |
| `VER`   | 1 byte     | Версия протокола.                                |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов. |
| `DTYP`  | 1 byte     | Тип запрашиваемых данных (смотри ниже).          |
| `KEY`   | 32 bytes   | DHT-ключ данных.                                 |

#### Типы данных для извлечения

| Значение | Тип данных        |
|----------|-------------------|
| `'I'`    | `Индексный пакет` |
| `'E'`    | `Почтовый пакет`  |
| `'C'`    | `Контакт`         |

### 3.2 Запрос удаления

Запрос к узлу на возврат `Информационного пакета удаления` для предоставленного DHT-ключа `Почтового пакета`.

Используется для определения доставки почты.

`Пакет ответа` с корректным статусом ожидается в ответ.  
`Пакет ответа` должен содержать `Информационный пакет удаления`.

| Поле    | Размер     | Описание                                         |
|---------|------------|--------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`.      |
| `TYPE`  | 1 byte     | Значение = `'Y'`.                                |
| `VER`   | 1 byte     | Версия протокола.                                |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов. |
| `KEY`   | 32 bytes   | DHT-ключ `Почтового пакета`.                     |

### 3.3 Запрос на хранение

Запрос к DHT на сохранение данных.

`Пакет ответа` с корректным статусом ожидается в ответ.

| Поле    | Размер     | Описание                                         |
|---------|------------|--------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`.      |
| `TYPE`  | 1 byte     | Значение = `'S'`.                                |
| `VER`   | 1 byte     | Версия протокола.                                |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов. |
| `HLEN`  | 2 bytes    | Длина поля `HK`.                                 |
| `HK`    | byte[]     | HashCash (`HLEN` байт).                          |
| `DLEN`  | 2 bytes    | Длина поля `DATA`.                               |
| `DATA`  | byte[]     | `Data Packet` to store (`DLEN` bytes). Can be an `Index Packet`, `Email Packet`, or `Email Destination`. |

### 3.4 Запрос на удаление почтового пакета

Запрос на удаление `Почтового пакета` по DHT-ключу.

`Пакет ответа` с корректным статусом ожидается в ответ.

| Поле    | Размер     | Описание                                         |
|---------|------------|--------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`.      |
| `TYPE`  | 1 byte     | Значение = `'D'`                                 |
| `VER`   | 1 byte     | Версия протокола.                                |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов. |
| `KEY`   | 32 bytes   | DHT-ключ почтового пакета к удалению.            |
| `DA`    | 32 bytes   | `Авторизация Удаления` (SHA-256 должен соответствовать значению `Подтверждения Удаления`) |

### 3.5 Запрос на удаления индекса

Запрос на удаление одной и более записей (ключей `Почтовых пакетов`) из хранимого `Индексного пакета`.

`Пакет ответа` с корректным статусом ожидается в ответ.

| Поле    | Размер     | Описание                                                                                                                   |
|---------|------------|----------------------------------------------------------------------------------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`.                                                                                |
| `TYPE`  | 1 byte     | Значение = `'X'`                                                                                                           |
| `VER`   | 1 byte     | Версия протокола.                                                                                                          |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов.                                                                           |
| `DH`    | 32 bytes   | SHA-256 хэш Почтового назначения (имя Индексного пакета)                                                                   |
| `N`     | 1 byte     | Число записей                                                                                                              |
| `DHT1`  | 32 bytes   | Первый DHT-ключ на удаление                                                                                                |
| `DA1`   | 32 bytes   | `Авторизация Удаления` (SHA-256 должен соответствовать значению `Подтверждения Удаления` из `Почтового пакета` для `DHT1`) |
| `DHT2`  | 32 bytes   | Второй DHT-ключ на удаление                                                                                                |
| `DA2`   | 32 bytes   | `Авторизация Удаления` (SHA-256 должен соответствовать значению `Подтверждения Удаления` из `Почтового пакета` для `DHT2`) |
| ...     | ...        | ...                                                                                                                        |
| `DHTn`  | 32 bytes   | n-ый DHT-ключ на удаление                                                                                                  |
| `DAn`   | 32 bytes   | `Авторизация Удаления` (SHA-256 должен соответствовать значению `Подтверждения Удаления` из `Почтового пакета` для `DHTn`) |

### 3.6 Запрос ближайших узлов

Запрос узлов, ближайших к переданому DHT-ключу.

`Пакет ответа` с корректным статусом ожидается в ответ.

| Поле    | Размер     | Описание                                         |
|---------|------------|--------------------------------------------------|
| `PFX`   | 4 bytes    | Префикс, должен быть `0x6D 0x30 0x52 0xE9`.      |
| `TYPE`  | 1 byte     | Значение = `'F'`                                 |
| `VER`   | 1 byte     | Версия протокола.                                |
| `CID`   | 32 bytes   | Идентификатора сессии, используется для ответов. |
| `KEY`   | 32 bytes   | DHT-ключ                                         |
