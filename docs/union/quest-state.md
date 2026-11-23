# Чтение состояния квестов

Пример из плагина для русской «Одиссеи» 2.7.5.2. Работа с парсером и журналом относится к Gothic II «Ночь Ворона». Имена тем, флаги и условия диалогов относятся к скриптам мода.

## Связь темы с фактом в игре

1. Найди `TOPIC_*` и обработчик, в котором создается тема через `Log_CreateTopic` с типом `LOG_MISSION`. Обработчик может быть диалогом, триггером или использованием предмета.
2. В плагине получи имя темы из `zCParser` по символу `TOPIC_*`. Проверяй тип и количество значений перед чтением. Для числовых флагов используй `std::optional<int>`, чтобы сохранить различие между отсутствующим символом и существующим флагом со значением `0`.
3. Сопоставь заголовок с записью в `oCLogManager`. Храни статус журнала и результаты чтения флагов отдельно. Если тема еще не появилась, проверь условия ее открытия в скриптах выдачи задания.
4. Преобразуй данные в статус отдельной функцией. Полезно тестировать ее обычным C++ без запущенной игры, в том числе на отсутствующих и противоречивых данных. [Чтение полей и журнала](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/180e38e05d50d66dcf1ba702e3c10e4dd01b9fab/src/Plugin.ipp#L70-L153) · [Тесты состояний](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/tree/180e38e05d50d66dcf1ba702e3c10e4dd01b9fab/tests)

Пример чтения скалярного флага для `.ipp` в `Gothic_II_Addon` с подключенным `<optional>` и загруженным парсером игры.

```cpp
std::optional<int> ReadScriptInt(zCParser& parser, const char* name) {
    auto* symbol = parser.GetSymbol(name);
    if (!symbol || symbol->type != zPAR_TYPE_INT || symbol->ele != 1
        || symbol->HasFlag(zPAR_FLAG_CLASSVAR)) {
        return std::nullopt;
    }

    int value = 0;
    symbol->GetValue(value, 0);
    return value;
}
```

## Главы, визиты и локальные этапы

Идентификатор темы сохраняется между мирами, а доступные шаги могут меняться. Например, «Флот орков» проходит через несколько островов и кораблей. Если задача требует процент перед отплытием, считай проверяемый местный этап и отдельно учитывай завершение всей цепочки. `LOG_NOTE` оставляй в разделе заметок, если пользователь просит каталог миссий. [Пример правил прогресса](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/180e38e05d50d66dcf1ba702e3c10e4dd01b9fab/docs/departure-progress.md) · [Пример сверки каталога со скриптами](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/fbc2ba82168fc5febec7b702d8b8fe1d7c960797/docs/chapter-2-audit.md)
