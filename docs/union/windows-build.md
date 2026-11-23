# Сборка и упаковка Union-плагина

Рецепт проверен в GitHub Actions для Gothic II «Ночь Ворона» на Windows x86. Пример использует Visual Studio 2022, CMake и Git.

| Зависимость | Ревизия примера |
| --- | --- |
| Union plugin template | `42a8a0cb61a5390696cdbcfe1a918c9e1b25b6f8` |
| Gothic API | `6e554b0acb199c9544037fb7136e07fad44a2fc4` |
| Union API | `167c5573653a8d89c1fee01865b1ac4502935e3f` |

## Воспроизведение сборки

```powershell
git clone https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector.git
Set-Location OdysseeQuestInspector
git checkout --detach 180e38e05d50d66dcf1ba702e3c10e4dd01b9fab
.\Build-Windows.ps1
```

Скрипт выбирает закрепленную ревизию шаблона, обновляет его подмодули и собирает конфигурацию `Release` с `-G "Visual Studio 17 2022" -A Win32 -DGOTHIC_API_G2A=ON`. Результат в `release/OdysseeQuestInspector.vdf`. [Скрипт сборки](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/blob/180e38e05d50d66dcf1ba702e3c10e4dd01b9fab/Build-Windows.ps1)

Внутри VDF примера DLL плагина и `UnionAPI.dll` находятся в `SYSTEM/AUTORUN`, а скомпилированная текстура в `_WORK/DATA/TEXTURES/_COMPILED`. При переносе в другой проект сверяй содержимое с его зависимостями и ресурсами. При закрытой игре установи VDF в `Gothic II/Data` и запусти игру с Union. [Рабочий прогон сборки и упаковки](https://github.com/piratvasyamorexod-alt/OdysseeQuestInspector/actions/runs/45122695698)
