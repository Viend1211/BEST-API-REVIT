# RSN Models Mini — плагин для Revit 2022 и Revit Server (BEST API)

> Быстрый инструмент для инвентаризации, резервного копирования, загрузки и обслуживания моделей на Revit Server. Никакой магии — только польза.


## ✨ Возможности

* **Парсинг Revit Server**: находит модели на выбранном сервере и выгружает список в `.txt` с полными путями.
* **Бэкапы через RevitServerTool**: на базе выгрузки генерируется `.bat` для планового скачивания локальных копий (папки — с датой/временем).
* **Загрузка на Revit Server из локальных RVT**: опционально сохраняет исходную структуру папок.
* **Пакетное подключение связей** *(WIP)*: добавляет связи в открытый проект с автоотключением ненужных рабочих наборов.
* **Изменение рабочих наборов у загруженных файлов**: правит рабочие наборы уже размещённых связей.
* **Сравнение «сервер ↔ локальная папка»**: помогает обновлять модели на сервере.

Плагин доступен из **Настройки → Внешние инструменты**. При желании можно вынести в отдельную вкладку (см. ниже раздел «Установка»).

---

## 📦 Установка

### 1) Размещение файлов

* Скопируйте `ModelsMini.dll` (и остальные файлы плагина) в удобную вам папку (например, `C:\Tools\RSNModelsMini`).
* Поместите `.addin` в одну из типовых директорий Revit:

  * `%ProgramData%\Autodesk\Revit\Addins\2022`
  * или `%AppData%\Autodesk\Revit\Addins\2022`

### 2) Вариант с отдельной вкладкой (Ribbon)

Добавьте в `.addin` секцию **Application**:

```xml
<AddIn Type="Application">
  <Name>RSNModelsMiniApp</Name>
  <Assembly>"ПУТЬ_К_ФАЙЛУ"\ModelsMini.dll</Assembly>
  <AddInId>4c74d634-3c63-4306-a3d0-111122223333</AddInId>
  <FullClassName>RSNModelsMini.App</FullClassName>
  <VendorId>COMP</VendorId>
  <VendorDescription>Ваша компания или имя</VendorDescription>
</AddIn>
```

После перезапуска Revit появится отдельная вкладка/кнопка.

---

## 🚀 Быстрый старт (бэкапы через RevitServerTool)

1. В плагине **просканируйте сервер** и **сохраните TXT** со списком моделей (полные пути на Revit Server).
2. Составьте `.bat` (см. примеры ниже) и проверьте запуск вручную.
3. Запланируйте выполнение через «Планировщик заданий» (Task Scheduler).

> **Важно:** пути с кириллицей? В начале `.bat` добавьте кодировку: `chcp 65001`.

---

## 🧰 Пример `.bat` для скачивания локальных копий

> Пример ниже иллюстративный. Замените IP/пути/имена на свои. Путь к `RevitServerTool.exe` зависит от версии Revit.

chcp 65001
set "rvtservtoolpath=C:\Program Files\Autodesk\Revit 2022\RevitServerToolCommand\RevitServerTool.exe"
set "rvtserv=192.168.88.21"
set "modelpath=APARTMENT_SUKHAREVSKAYA\04_SS\"
set "targetdir=L:\Revit_Backup_SUKHAREVSKAYA\SS"
set "AR_B2=PHANTOM_MDC_R22_FA_МС6.rvt"
set "AR_B4=PHANTOM_MDC_R22_FA_МС8.rvt"
set "AR_B5=PHANTOM_MDC_R22_LAN_МС6.rvt"
set "AR_B6=PHANTOM_MDC_R22_LAN_МС8.rvt"
set "AR_B7=PHANTOM_MDC_R22_SS_МС6.rvt"
set "AR_B8=PHANTOM_MDC_R22_SS_МС8.rvt"

for /f "tokens=2 delims==" %%a in ('wmic OS Get localdatetime /value') do set "dt=%%a"
set "YYYY=%dt:~0,4%" & set "MM=%dt:~4,2%" & set "DD=%dt:~6,2%"
set "datestamp=%YYYY%%MM%%DD%"
set "targetfolder=%targetdir%\%datestamp%\"
md "%targetfolder%" 2>nul

"%rvtservtoolpath%" createLocalRVT "%modelpath%%AR_B2%" -s %rvtserv% -d "%targetfolder%%AR_B2%" -o
"%rvtservtoolpath%" createLocalRVT "%modelpath%%AR_B4%" -s %rvtserv% -d "%targetfolder%%AR_B4%" -o
"%rvtservtoolpath%" createLocalRVT "%modelpath%%AR_B5%" -s %rvtserv% -d "%targetfolder%%AR_B5%" -o
"%rvtservtoolpath%" createLocalRVT "%modelpath%%AR_B6%" -s %rvtserv% -d "%targetfolder%%AR_B6%" -o
"%rvtservtoolpath%" createLocalRVT "%modelpath%%AR_B7%" -s %rvtserv% -d "%targetfolder%%AR_B7%" -o
"%rvtservtoolpath%" createLocalRVT "%modelpath%%AR_B8%" -s %rvtserv% -d "%targetfolder%%AR_B8%" -o

echo Finished
exit /b 0
```

---

## 🔁 Генерация `.bat` по TXT-списку

Если плагин выгрузил список моделей **по одной на строку** (полный путь на Revit Server), можно собрать универсальный `.bat`.

```bat
@echo off
chcp 65001 >nul
set "rvtservtoolpath=C:\Program Files\Autodesk\Revit 2022\RevitServerToolCommand\RevitServerTool.exe"
set "rvtserv=192.168.88.21"
set "targetdir=D:\Revit_Backups"
set "models_list=models.txt"   rem одна модель на строку, пример: APARTMENT\03_PL\PHANTOM_MDC_R22_PT_МС6.rvt

for /f "tokens=2 delims==" %%a in ('wmic OS Get localdatetime /value') do set "dt=%%a"
set "YYYY=%dt:~0,4%" & set "MM=%dt:~4,2%" & set "DD=%dt:~6,2%"
set "datestamp=%YYYY%%MM%%DD%"
set "targetfolder=%targetdir%\%datestamp%\"
md "%targetfolder%" 2>nul

for /f "usebackq delims=" %%P in ("%models_list%") do (
  rem %%P = APARTMENT\03_PL\PHANTOM...rvt
  for %%# in ("%%P") do set "_fname=%%~nx#"
  call "%rvtservtoolpath%" createLocalRVT "%%P" -s %rvtserv% -d "%targetfolder%!_fname!" -o
)

echo Done
exit /b 0
```

> При необходимости группируйте модели по разделам и создавайте отдельные `targetdir`.

---

## ⬆️ Загрузка на Revit Server из локальных RVT

* Поддерживается пакетная загрузка с **сохранением структуры папок** (опционально).
* Типовой сценарий: указать корневую локальную папку и целевой узел на сервере.
* При загрузке создаётся/назначается корректный **рабочий набор** для связей (если не указан — создаётся «Связи» по умолчанию).

---

## 🔗 Пакетное добавление связей *(WIP)*

* Добавляет несколько RVT в **текущий проект**.
* Может **автоматически отключать ненужные рабочие наборы** при подключении.
* Статус: в доработке (см. Roadmap).

---

## 🧱 Управление рабочими наборами у загруженных файлов

* Для уже размещённых связей можно массово изменить **целевой рабочий набор**.
* Полезно для приведения проекта к корпоративному стандарту.

---

## ⚖️ Сравнение «сервер ↔ локальная папка»

* Плагин сравнивает состав моделей на сервере и в локальном хранилище.
* Помогает быстро загрузить обновленный rvt на сервер.

---

## 🐞 Тонкости и ограничения

* `RevitServerTool.exe` должен соответствовать **той же версии Revit**, что и модели.
* Пути с кириллицей: используйте `chcp 65001` в начале `.bat`.
* `wmic` на новых системах помечен как устаревший, но всё ещё работает; при желании можно заменить на PowerShell.
* Сетевые диски должны быть **подключены** на момент запуска задания.

---
## ❓ FAQ

**Где искать TXT-выгрузку?**
— В папке, которую укажете при сохранении. Формат — одна модель на строку, полный путь на Revit Server.

**Почему не качает?**
— Проверьте доступ к серверу, корректность `modelpath`, совпадение версии `RevitServerTool`, и права на целевую папку.

**Можно ли сделать отдельную вкладку?**
— Да, добавьте секцию `<AddIn Type="Application">` в `.addin` (см. «Установка»).


---

**P.S.** Если что-то пойдёт не так — пишите. Сделаем, чтобы «работало и не отвлекало». 💪
