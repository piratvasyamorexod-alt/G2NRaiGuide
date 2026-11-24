# Игровой интерфейс, пауза и ввод

Проверенный пример — Odyssee Quest Inspector 1.0.7 для Gothic II «Ночь Ворона» и Union. [Сборка и зависимости](windows-build.md) · [Мышь на паузе](mouse-input.md)

## Окно с паузой

Журнал реализован как `zCView`, добавленный в `screen`. Окно открывается из игрового режима, когда доступны `ogame`, `screen`, шрифт и ввод и когда игра еще не на паузе. Конструктор вызывает `oCGame::Pause()` и регистрирует виджет через `SetEnableHandleEvent(TRUE)` и `SetHandleEventTop()`.

`HandleEvent()` накапливает действия. В проверенном варианте журнал обрабатывает их после штатного `zCInputCallback::GetInput()` по адресу `0x007A55C0` для G2A 2.6 fix. Цикл может открыть журнал после опроса клавиш, даже когда окно еще не создано. Хук `oCGame::Render()` обновляет окно как запасной путь, если штатный опрос пропущен. Между двумя хуками передается признак уже выполненного обновления. [Опрос ввода и отрисовка](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L1242-L1262)

Закрывай окно в цикле после возврата из обработчика событий. Отключи обработчик, убери окно из родителя, верни системе курсор и захват мыши, затем освободи паузу, взятую при открытии. Выполняй тот же путь очистки при загрузке, смене мира и завершении игры. [Открытие и закрытие окна](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L495-L520) · [Очистка при смене игры](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L1220-L1297)

Фрагмент для пространства имен `Gothic_II_Addon` в Union-плагине. Создавай `ModalView` при доступном `screen` со шрифтом и при `game.IsOnPause() == false`. В цикле перед отрисовкой удаляй окно, когда выставлен `closeRequested`.

```cpp
class ModalView : public zCView {
    oCGame& game;

public:
    bool closeRequested = false;

    explicit ModalView(oCGame& session) : zCView(296, 610, 7600, 6520), game(session) {
        SetFont(screen->GetFont());
        game.Pause();
        screen->InsertItem(this);
        SetEnableHandleEvent(TRUE);
        SetHandleEventTop();
    }

    ~ModalView() override {
        SetEnableHandleEvent(FALSE);
        if (owner) owner->RemoveItem(this);
        if (game.IsOnPause()) game.Unpause();
    }

    int HandleEvent(int key) override {
        if (key == KEY_ESCAPE) closeRequested = true;
        return TRUE;
    }
};
```

## Открытие одним нажатием

Дай одной функции переключать журнал и в игре, и на паузе. Проверяй фронт нажатия `KEY_APOSTROPHE` через `KeyPressed()`, а в качестве запасного источника возьми старший бит `GetAsyncKeyState()` для виртуальной клавиши, полученной из сканкода через `MapVirtualKeyW`. Храни один признак предыдущего состояния для обоих источников. Это сохраняет отклик на первое нажатие, когда движок или другой обработчик уже забрал одноразовый `KeyToggled()`. Пока клавиша удерживается, повторное обновление журнала не должно закрывать окно. [Переключение по апострофу](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L1111-L1151)

## Панели и прокрутка

У журнала отдельные области списка, статуса и карточки. Положение колонки статусов рассчитывается по ширине левой части, поэтому она остается на месте при показе подсказки справа. Скрывай подсказку при открытии окна и смене выбранного задания. Открывай ее явным действием, например Enter или нажатием кнопки. Для нескольких локаций дай кораблю собственную вкладку и при первом открытии выбирай вкладку по загруженному миру. [Вкладки и выбор мира](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L1121-L1217) · [Список и карточка](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L902-L1059)

Храни позицию списка и позицию текста отдельно. Направляй колесико в область под курсором. Для длинного текста вычисляй допустимое смещение как `max(0, всего строк - видимых строк)`. Высота бегунка зависит от доли видимых строк, положение — от смещения. При перетаскивании сохраняй точку захвата внутри бегунка. Для длинных подсказок также сохраняй клавиатурную прокрутку. [Бегунки](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L875-L965)

## Фон и текстуры

Используй одну нейтральную полупрозрачную текстуру для фона панели. Для фона, бегунков и резервного курсора в примере используются `zCView::Blit()` и `zRND_ALPHA_FUNC_BLEND`. Перед выводом сохраняй текущий источник альфы рендерера, устанавливай `zRND_ALPHA_SOURCE_MATERIAL` и возвращай прежнее значение после вывода. Для коротких подписей и текста поверх ярких участков используй тень и проверяй читаемость на светлом и темном фоне. [Виджеты и альфа](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L455-L510) · [Отрисовка панели](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L613-L637)

## Единый стиль игровых окон

Используй общий фон с рамкой и одинаковую палитру. В G2NRcharstats 0.1.1 и Odyssee Quest Inspector 1.1.0 совпадают данные текстуры панели и курсора. Обе панели используют `SetTransparency(248)` и смешивание по альфе материала. Названия выводятся цветом `0xFFF0DFBB`, основные значения — `0xFFFFEAC4`, вкладки и акценты — `0xFFDDB67D`, разделители — `0xFF655740`. Для текста применяется темная тень со сдвигом на один пиксель. Отдельные смысловые статусы можно выделять своим цветом. [Палитра окна характеристик](https://github.com/piratvasyamorexod-alt/G2NRcharstats/blob/059fe19279dbde91931c95abc672aee8ef06279e/src/Plugin.ipp#L14-L17) · [Фон и отрисовка инспектора](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/2edb51b36464777cfc55fb6933ea95235ca4697e/src/Plugin.ipp#L489-L515)

Ограничивай ширину окна долей ширины экрана и коэффициентом от его высоты. Позицию рассчитывай от центра, межстрочный шаг — от высоты игрового шрифта. Для `zCView(x1, y1, x2, y2)` передавай координаты двух углов в виртуальных единицах. У окна характеристик высота зависит от числа строк. Инспектор выше и шире, чтобы вместить список и карточку с подсказкой. Строй кликабельную область вкладки из тех же координат, что и подпись с подчеркиванием. [Размер окна характеристик](https://github.com/piratvasyamorexod-alt/G2NRcharstats/blob/059fe19279dbde91931c95abc672aee8ef06279e/src/status_layout.hpp#L15-L24) · [Размер и вкладки инспектора](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/2edb51b36464777cfc55fb6933ea95235ca4697e/src/Plugin.ipp#L404-L415) · [Зоны вкладок](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/2edb51b36464777cfc55fb6933ea95235ca4697e/src/Plugin.ipp#L852-L872)

В игре проверен вид окна характеристик в «Одиссее» и еще одной сборке Gothic II. Для инспектора 1.1.0 прошли сборка Windows, тесты статусов и тесты ввода. Вид этой версии инспектора в игре еще проверяется. [Сборка инспектора](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/actions/runs/45482482736)

## Русский шрифт

В русском шрифте, который использует журнал, отсутствуют глифы `Ё` и `ё`. Замени их в отображаемой копии строки перед вычислением ширины и переносами. Для сравнения названия с записью в игровом журнале сохраняй исходную строку. В примере `WideCharToMultiByte(1251, ...)` преобразует текст для `zSTRING`. [Преобразование и отрисовка текста](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/src/Plugin.ipp#L720-L783)

## Проверки

- Сборка 1.0.7 и тесты прошли. Локальный стенд проверил координаты, переключение вкладок и разделение системного и резервного курсоров. Windows-тест проверил прием движения при остановленном основном потоке, кнопки, колесико и возврат фокуса. Пользователь подтвердил работу и плавность в своей «Одиссее». [Сборка 1.0.7](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/actions/runs/45235472806)
- Для проверки нового окна открой его рядом с движущимся NPC и проверь, что мир приостановлен до закрытия. После закрытия мышью и Escape проверь возврат игрового управления. Повтори открытие после загрузки сохранения и перехода между мирами. [Сценарии проверки мыши](mouse-input.md#как-повторить-проверку)
