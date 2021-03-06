# Отладка под Windows

Если вы наблюдаете аварии или проблемы в работе Electron, которые, как вы считаете, вызваны самим Electron, а не приложением на JavaScript, отладка может быть немного сложной, особенно для разработчиков ранее не занимавшихся отладкой кода на C++. However, using Visual Studio, GitHub's hosted Electron Symbol Server, and the Electron source code, you can enable step-through debugging with breakpoints inside Electron's source code.

**See also**: There's a wealth of information on debugging Chromium, much of which also applies to Electron, on the Chromium developers site: [Debugging Chromium on Windows](https://www.chromium.org/developers/how-tos/debugging-on-windows).

## Требования

* **Отладочная сборка Electron**: Обычно проще всего собрать ее самостоятельно, используя инструменты и предварительные требования, перечисленные в [инструкции по сборке под Windows](build-instructions-windows.md). While you can attach to and debug Electron as you can download it directly, you will find that it is heavily optimized, making debugging substantially more difficult: The debugger will not be able to show you the content of all variables and the execution path can seem strange because of inlining, tail calls, and other compiler optimizations.

* **Visual Studio с инструментами C++**: бесплатная общественная редакция Visual Studio, можно использовать версии VS2013 и VS2015. После установки, [настройте Visual Studio для использования сервера символов Electron на GitHub](setting-up-symbol-server.md). Это позволит Visual Studio получить лучшее представление о том, что происходит внутри Electron, что позводит представить переменные в удобочитаемом формате.

* **ProcMon**: [бесплатный инструмент от SysInternals](https://technet.microsoft.com/en-us/sysinternals/processmonitor.aspx), позволяющий вам просматривать параметры процессов, файловые дескрипторы, и операции над реестром.

## Подключение к Electron для отладки

Для запуска сеанса отладки, откройте PowerShell/CMD и запустите отладочную сборку Electron, указав приложение в качестве параметра.

```powershell
$ ./out/D/electron.exe ~/my-electron-app/
```

### Задание точек останова

Затем, откройте Visual Studio. Electron не был собран из Visual Studio, и поэтому код не содержит файла проекта; тем не менее, вы можете открывать исходные файлы просто "как файл", то есть Visual Studio откроет их сами по себе. Тем не менее, вы можете ставить точки останова - Visual Studio автоматически определит, что этот исходный код соответствует исполняемому коду в подключенном процессе, и остановится на указанной точке останова.

Relevant code files can be found in `./atom/` as well as in Brightray, found in `./brightray/browser` and `./brightray/common`.

### Подключение

Вы можете подключить отладчик Visual Studio к запущенному процессу, на локальном или удаленном компьютере. После того как процесс был запущен, нажмите Debug / Attach to Process (или нажмите `CTRL + ALT + P`), чтобы открыть диалоговое окно «Attach to Process». Вы можете использовать эту возможность для отладки приложений, выполняемых на локальном или удаленном компьютере, и для одновременной отладки нескольких процессов.

Если Electron выполняется под учетной записью другого пользователя, установите флажок `Show processes from all users`. Обратите внимание, что вы увидите несколько процессов, их количество зависит от того, сколько BrowserWindows открыто в вашем приложении. Типичное одноокнонное приложение будет выглядеть в Visual Studio как два процесса `Electron.exe` - один для основного процесса и один для процесса визуализации. Поскольку список показывает вам только имена, в настоящее время нет надежного способа выяснить какой из них к чему относится.

### К какому процессу я должен подключиться?

Код, выполняющийся в рамках основного процесса (то есть, код находящийся в вашем основном JavaScript файле или вызывающийся из него), а также код, вызываемый с помощью remote (`require('electron').remote`), будет выполняться внутри основного процесса, в то время как остальной код будет выполняться внутри соответствующего процесса визуализации.

Вы можете подключиться к несколько программам для отладки, но только одна программа будет активна в отладчике в каждый момент времени. Вы можете задать активную программу в панели инструментов `Debug Location` либо в окне `Processes`.

## Использование ProcMon для наблюдения за процессом

В то время как Visual Studio отлично подходит для изучения конкретных путей выполнения, ProcMon действительно силен в наблюдении за всем, что делает ваше приложение с операционной системой - включая файлы, реестр, сеть, процессы и детальное профилирование процессов. Он пытается протоколировать **все** происходящие события, что может быть даже весьма излишним, но если вы стремитесь понять, что и как делает ваше приложение с операционной системой, это может быть ценным ресурсом.

В качестве введения в базовые и расширенные возможности отладки через ProcMon можно рекомендовать [это видео руководство](https://channel9.msdn.com/shows/defrag-tools/defrag-tools-4-process-monitor) от Microsoft.