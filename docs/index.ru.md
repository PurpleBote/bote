# Протокол I2P Bote

* [clearnet](https://bote.readthedocs.io/en/latest/)
* [I2P](http://polistern.i2p/bote/)

**I2P-Bote** - это бессерверный, основанный на [распределённой хеш-таблице Kademlia](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D1%81%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D1%91%D0%BD%D0%BD%D0%B0%D1%8F_%D1%85%D0%B5%D1%88-%D1%82%D0%B0%D0%B1%D0%BB%D0%B8%D1%86%D0%B0), протокол обмена зашифрованными сообщениями электронной почты.

## Версии

- [v5](v5/index.md): текущая версия с исправлеными проблемами коммуникации.
- [v6](v6/index.md): возможная будущая версия. В состоянии черновика.

### Устаревшие

- [v4](old/v4/introduction.md): не поддерживает новые I2P адреса.

## Реализации

- [i2p.i2p-bote](https://github.com/i2p/i2p.i2p-bote)
    - Оригинальная реализация протокола в виде плагина для Java I2P роутера
    - Версии протокола: v4
    - Статус: выглядит заброшенным
- [i2pboted](https://github.com/majestrate/i2pboted)
    - Самостоятельный клиент на языке Go
    - Версии протокола: v4
    - Статус: выглядит заброшенным
- [pboted](https://github.com/PurpleBote/pboted)
    - Самостоятельный клиент на языке C++
    - Версии протокола: v4, v5
    - Статус: активен

## Ресурсы

- [Документация](https://bote.readthedocs.io/ru/latest/) ([I2P](http://polistern.i2p/bote/))
- [Заявки/Вопросы](https://github.com/polistern/bote/issues)

## Отправка изменений

Вы можете отправлять пулл-реквест в [документацию I2P Bote](https://github.com/PurpleBote/bote/pull/new/master) с ясным списком сделаных изменений (подробнее о [пулл-реквестах](http://help.github.com/pull-requests/)).

Пожалуйста, убедитесь:

- Ваши изменния атомарны (одно полноценное изменение на коммит).
- Ваши сообщения в описании коммитов понятны.  
  Однострочные сообщения подходят для малых изменний, но большие должны форматироваться следующим образом:

```bash
$ git commit -m "A brief summary of the commit
>
> A paragraph describing what changed and its impact."
```

## Пожертвования

Проект не имеет цели получения коммерческой выгоды.

- **XMR**: `85P3aEXrYMn1YxnQaZSBWy6Ur6j9PVRxmCd3Ey1UanKAdKnhd2iYNdrEhNJ2JeUdcC8otSHogRTnydn4aMh8DwbSMs4N13Z`

## Также

Если у Вас возникли вопросы по коду и оформлению, то Вы можете найти меня на канале `[#dev]` в IRC-сети **ilita** в I2P.

Искренне Ваша,  
*polistern*
