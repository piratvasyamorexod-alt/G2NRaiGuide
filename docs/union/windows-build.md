# Сборка и упаковка Union-плагина

Рецепт проверен в GitHub Actions на примере Odyssee Quest Inspector 1.0.7 для Gothic II «Ночь Ворона» на Windows x86. Пример использует Visual Studio 2022, CMake и Git.

| Зависимость | Ревизия примера |
| --- | --- |
| Union plugin template | `42a8a0cb61a5390696cdbcfe1a918c9e1b25b6f8` |
| Gothic API | `6e554b0acb199c9544037fb7136e07fad44a2fc4` |
| Union API | `167c5573653a8d89c1fee01865b1ac4502935e3f` |

## Воспроизведение сборки

```powershell
git clone https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector.git
Set-Location OdysseeQuestInspector
git checkout --detach c6cacbd167837afe84364069ecc25a29e97bca81
.\Build-Windows.ps1
```

Скрипт выбирает закрепленную ревизию шаблона, обновляет его подмодули и собирает конфигурацию `Release` с `-G "Visual Studio 17 2022" -A Win32 -DGOTHIC_API_G2A=ON`. Результат в `release/OdysseeQuestInspector.vdf`. Номер версии из сценария попадает в ресурс DLL. При выпуске синхронизируй его с номером, который выводит журнал. [Скрипт сборки](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/Build-Windows.ps1)

Внутри VDF версии 1.0.7 находятся DLL плагина и `UnionAPI.dll` в `SYSTEM/AUTORUN`, а также три текстуры для панели, бегунка и резервного курсора в `_WORK/DATA/TEXTURES/_COMPILED`. Если берешь сборку из Actions, сверяй SHA коммита, версию DLL и содержимое VDF. При закрытой игре замени предыдущий VDF в той папке, откуда его загружает Union. В пользовательской установке это был `Gothic II\Data\Plugins`. После запуска сверь номер в углу журнала. [Артефакт и Windows-прогон](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/actions/runs/45235472806)

## Windows-тест мыши

Из корня репозитория плагина выполни команды в Windows. Тест компилирует тот же обработчик мыши, который использует плагин.

```powershell
cmake -S tests/win32 -B win32-mouse-test -G "Visual Studio 17 2022" -A Win32
cmake --build win32-mouse-test --config Release
ctest --test-dir win32-mouse-test -C Release --output-on-failure
```

Тест открывает окно и отправляет ввод через `SendInput`. Он проверяет доставку движения, клика и прокрутки, в том числе движение при остановленном основном потоке. Связку с ZenGin, отрисовку и плавность проверяй в самой игре. [Исходник теста](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/c6cacbd167837afe84364069ecc25a29e97bca81/tests/win32/journal_mouse.cpp) · [Сценарии проверки](mouse-input.md#как-повторить-проверку)
