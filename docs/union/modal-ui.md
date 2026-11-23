# Игровой интерфейс, пауза и ввод

Пример для Gothic II «Ночь Ворона» и Union. Точка входа для сборки и версии зависимостей описаны в [рецепте сборки](windows-build.md).

## Окно с паузой

Журнал реализован как `zCView`, добавленный в `screen`. Окно открывается из игрового режима, когда доступны `ogame`, `screen`, шрифт и ввод и когда игра еще не на паузе. Конструктор вызывает `oCGame::Pause()` и регистрирует виджет через `SetEnableHandleEvent(TRUE)` и `SetHandleEventTop()`.

`HandleEvent()` складывает действия в состояние, а хук `oCGame::Render()` обрабатывает их и обновляет окно перед обычной отрисовкой. Это дает журналу возможность реагировать на ввод во время паузы. Обрабатывай закрытие в цикле, после выхода из обработчика событий. При закрытии отключай обработчик, убирай окно из родительского `zCView` и освобождай паузу, взятую при открытии. Выполняй тот же путь очистки при загрузке, смене мира и завершении игры. [Окно и его жизненный цикл](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/180e38e05d50d66dcf1ba702e3c10e4dd01b9fab/src/Plugin.ipp#L410-L544) · [Обновление и хуки](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/180e38e05d50d66dcf1ba702e3c10e4dd01b9fab/src/Plugin.ipp#L986-L1144)

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

## Курсор и прокрутка

`zinput->GetMousePos(dx, dy, wheel)` в проверенной версии G2A дает относительное движение. Накапливай `dx` и `dy` в координатах курсора. Для проверки клика переведи экранное положение в локальные координаты окна, вычтя `GetPixelPos()` и преобразовав результат через `anx()` и `any()`. Конвертируй координаты обратно с `nax()` и `nay()` при отрисовке поверх окна в экранных пикселях.

Клик определяется по переходу левой кнопки из отпущенной в нажатую. Храни позицию списка и позицию текста отдельно. Направляй колесико в область под курсором. Для длинного текста вычисляй допустимое смещение как `max(0, всего строк - видимых строк)`. Положение бегунка пропорционально смещению. При перетаскивании сохраняй точку захвата внутри бегунка, чтобы при первом движении он оставался под курсором. [Обработка мыши](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/180e38e05d50d66dcf1ba702e3c10e4dd01b9fab/src/Plugin.ipp#L479-L522) · [Прокрутка подсказки](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/180e38e05d50d66dcf1ba702e3c10e4dd01b9fab/src/Plugin.ipp#L766-L846)

## Русский шрифт

В русском шрифте, который использует журнал, отсутствуют глифы `Ё` и `ё`. Замени их в отображаемой копии строки перед вычислением ширины и переносами. Для сравнения названия с записью в игровом журнале сохраняй исходную строку. В примере `WideCharToMultiByte(1251, ...)` преобразует текст для `zSTRING`. [Преобразование и отрисовка текста](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/180e38e05d50d66dcf1ba702e3c10e4dd01b9fab/src/Plugin.ipp#L611-L670)

## Проверки

- Windows-сборка с мышью прошла, VDF проверен. Локальный стенд с подставным вводом подтвердил переключение вкладок, выбор строк, независимую прокрутку, перетаскивание и снятие паузы при закрытии. [Сборка](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/actions/runs/45122695698)
- Для проверки в игре открой окно рядом с движущимся NPC. Проверь игровые часы, клавиатуру, мышь и бегунок, затем закрой окно мышью и клавиатурой. Проверь возврат управления после закрытия и загрузку при открытом окне. Совместимость курсора в самой игре ожидает подтверждения.
