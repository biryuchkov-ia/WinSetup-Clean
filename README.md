# Сценарий автоматической установки Windows (autounattend.xml)

---

## 🇷🇺 Русская версия

Этот репозиторий содержит кастомизированный, оптимизированный и полностью очищенный файл `autounattend.xml`, предназначенный для автоматизации установки Windows 10/11. **Скрипт разработан специально для игровых ПК/ноутбуков, чтобы обеспечить максимальную производительность,** создавая быструю, отзывчивую и минималистичную операционную систему, уменьшая количество предустановленного ПО (bloatware), отключая фоновые процессы и ненужные облачные интеграции.

Сценарий автоматизирует многие шаги установки, позволяя при этом пользователю контролировать критически важные решения, такие как выбор диска.

## 🎯 Философия проекта

1.  **Производительность и контроль**: Активация высокопроизводительных режимов, жесткая оптимизация электропитания CPU, полное отключение фоновой активности UWP-приложений и улучшение использования ресурсов.

2.  **Минимализм, чистота и отсутствие ИИ**: Полное удаление и блокировка всех ИИ-функций (**Microsoft Copilot, Cortana, Bing Search, Windows Recall / ИИ-анализ экрана**). Удаление нежелательных UWP-приложений, OneDrive, Microsoft Teams/Chat и других рекламных элементов. Очистка рабочего стола (удаление ярлыка Edge).

3.  **Приватность**: Значительное сокращение сбора данных (телеметрии), отключение ряда облачных сервисов и нежелательных функций.

4.  **Управляемая автоматизация**: Большинство шагов автоматизированы, но пользователь сохраняет контроль над ключевыми этапами, такими как ручной выбор раздела диска.

## 📦 Что останется в системе при первом входе?

Поскольку скрипт направлен на тотальную очистку, после первого входа в систему вы получите абсолютно чистую операционную систему.

**✅ В системе останутся:**

*   **Защитник Windows (Windows Security)** - единственная программа в автозагрузке для обеспечения базовой безопасности.

*   **Microsoft Edge** - стандартный браузер для скачивания нужных программ (аккуратно закреплен на панели задач, ярлык с рабочего стола удален).

*   **Microsoft Store** - официальный магазин (закреплен на панели задач) для ручной установки только нужных вам UWP-приложений.

*   **Базовые системные утилиты** - Проводник, Блокнот, Калькулятор, Ножницы, Paint и стандартные средства ОС.

**🚫 В системе будут полностью отсутствовать:**

*   **Автоматически скачанные драйверы** (Windows Update не будет ставить драйверы без вашего ведома, вам нужно будет установить их вручную с официального сайта производителя).

*   **Мусорные приложения** (TikTok, Instagram, Spotify, Candy Crush и т.д.).

*   **OneDrive, Microsoft Teams, Cortana, Copilot и Recall**.

*   **Фоновые UWP-процессы** (все они жестко заблокированы политикой `Force Deny`).

## ⚙️ Глубокий анализ: Как это работает?

Файл `autounattend.xml` - это набор инструкций для программы установки Windows, выполняемых на различных этапах, называемых "проходами" (passes).

### Проход 1: `Windows PE` (Среда предустановки Windows)

Выполняется сразу после загрузки с установочного носителя.

*   **Ручной выбор диска**: Автоматическое форматирование или выбор диска не производится. Пользователь сможет самостоятельно выбрать или настроить разделы диска.

*   **Принятие EULA**: Экран с лицензионным соглашением **отображается** для ручного принятия.

*   **Обход системных требований**: В реестр среды предустановки вносятся ключи, отключающие проверку на наличие TPM 2.0, Secure Boot и достаточного объема ОЗУ.

### Проход 2: `Specialize` (Спецификация)

Основной этап кастомизации, где применяются настройки, уникальные для данной установки Windows.

*   **Извлечение скриптов**: Запускается PowerShell-скрипт, который "распаковывает" остальные скрипты из `autounattend.xml`.

*   **Запуск `Specialize.ps1`**: Этот основной PowerShell-скрипт выполняет большинство системных настроек:

    *   **Отключение ИИ-функций**: Глобально отключает Cortana, Windows Copilot и функцию ИИ-анализа экрана (Windows Recall).

    *   **Отключение автоматической установки драйверов**: Запрещает Windows Update автоматически загружать и устанавливать драйверы устройств.

    *   **Оптимизация производительности**:
        *   Отключение гибернации и зарезервированного хранилища.
        *   Включение `CompactOS` для сжатия системных файлов.
        *   Глобальное отключение фоновых приложений UWP (`LetAppsRunInBackground = 2`).
        *   Отключение Power Throttling, Network Throttling, установка System Responsiveness в 0.
        *   Отключение Game DVR на уровне системы.
        
    *   **Настройка схемы питания "Максимальная производительность"**:
        *   **CPU:** Режим Turbo Boost установлен на `Aggressive`, парковка ядер отключена.
        *   **PCI Express:** Link State Power Management полностью отключен.
        *   **USB:** Выборочная приостановка отключена.
        *   **Wi-Fi:** Адаптер переведен в режим максимальной производительности.

    *   **Отключение телеметрии**: Останавливает и отключает службу сбора диагностических данных `DiagTrack`.

    *   **Очистка системы**: Удаление OneDrive, ярлыка Microsoft Edge с рабочего стола и более 20 предустановленных UWP-приложений.

    *   **Настройка интерфейса**:
        *   Применяет кастомный макет панели задач.
        *   Полностью очищает меню "Пуск" от плиток.
        *   Устанавливает метку системного диска `C:` как "Windows".

### Проход 3: `OobeSystem` (Out-of-Box Experience)

*   **Пропуск OOBE-экранов**: Скрываются экраны первоначальной настройки и назойливое предложение войти в учетную запись Microsoft.

### Задачи после установки (FirstLogonCommands)

Выполняются один раз при первом входе нового пользователя (`UserOnce.ps1`).

*   **Отключение акселерации мыши** ("Повышенная точность указателя").
*   **Отключение Game Bar** на уровне пользователя.
*   **Настройка Проводника**: Устанавливает стартовой папкой "Этот компьютер".
*   **Улучшение качества JPEG**: Устанавливает качество сохранения JPEG в 100%.

## 📋 Как использовать

1.  **Скачайте** файл `autounattend.xml` из этого репозитория.

2.  **Создайте загрузочный USB-носитель** с официальным образом Windows 10 или 11.

3.  **Поместите `autounattend.xml`** в корень этого USB-носителя.

4.  **Загрузите компьютер** с данного USB-носителя.

5.  Процесс установки будет автоматизирован. Вам нужно будет вручную принять EULA и выбрать диск для установки. Остальные шаги будут выполнены автоматически.

> **‼️ ПРИМЕЧАНИЕ:** Этот сценарий **не** форматирует диск автоматически. Вы сможете вручную выбрать и настроить дисковые разделы в процессе установки. Всегда создавайте резервную копию важных данных перед установкой операционной системы.

### Лицензия / Отказ от ответственности

Этот проект распространяется под лицензией **MIT** — подробности см. в файле [LICENSE](LICENSE).

### ⚠️ ВАЖНО:

* **Использование на свой страх и риск:** Данный файл ответов и инструкция предоставляются «как есть» (as is). Автор не несет ответственности за любые сбои в работе вашей системы, потерю данных или повреждение оборудования.
* **Внешние зависимости:** При установке библиотек Вы используете стороннее ПО, за безопасность которого автор ответственности не несет.
* **Проверка кода:** Настоятельно рекомендуется изучить код перед запуском и протестировать его в безопасной среде, а перед установкой операционной системы всегда создавайте резервную копию важных данных.

---

**Авторские права © 2026 [Иван Бирючков / https://github.com/biryuchkov-ia]**

---

## 🇺🇸 English Version

This repository contains a customized, highly optimized, and clean `autounattend.xml` file designed to automate Windows 10/11 installations. **The script is specifically developed for gaming PCs/laptops to ensure maximum performance,** creating a fast, responsive, and minimalistic operating system by reducing bloatware, completely disabling background UWP apps, and removing unnecessary cloud integrations.

The script automates many installation steps, while allowing the user to retain control over critical decisions such as manual disk selection.

## 🎯 Project Philosophy

1.  **Performance and Control**: Activating high-performance modes, optimizing CPU power management, strictly disabling background activity for UWP apps, and enhancing disk space utilization.

2.  **Minimalism, Cleanliness, and AI-Free**: Complete removal and blocking of all Microsoft AI features (**Microsoft Copilot, Cortana, Bing Search, Windows Recall / AI Data Analysis**). Removing unwanted UWP applications, OneDrive, Microsoft Teams/Chat, and other promotional elements. Desktop cleanup (removing the Edge shortcut).

3.  **Privacy**: Significantly reducing data collection (telemetry), disabling a range of cloud services and undesirable features.

4.  **Managed Automation**: Most steps are automated, but the user retains control over key stages, such as disk partition selection.

## 📦 What remains in the system upon first login?

Since the script is heavily focused on debloating, you will get an absolute clean slate upon your first login.

**✅ The system WILL HAVE:**

*   **Windows Security (Defender)** - the only program in the startup items to ensure basic protection.

*   **Microsoft Edge** - the default browser needed to download your preferred software (pinned to the taskbar, desktop shortcut removed).

*   **Microsoft Store** - the official store (pinned to the taskbar) for manual installation of only the UWP apps you actually need.

*   **Standard OS Utilities** - File Explorer, Notepad, Calculator, Snipping Tool, Paint, and standard Windows Settings.

**🚫 The system will DEFINITELY NOT HAVE:**

*   **Automatically downloaded drivers** (Windows Update will not install drivers behind your back; you must install them manually from the manufacturer's official website).

*   **Third-party bloatware** (TikTok, Instagram, Spotify, Candy Crush, etc.).

*   **OneDrive, Microsoft Teams, Cortana, Copilot, and Recall**.

*   **Background UWP processes** (all are strictly blocked by a `Force Deny` policy).

## ⚙️ Deep Dive: How It Works

The `autounattend.xml` file is a set of instructions for the Windows Setup program, executed during various stages called "passes."

### Pass 1: `Windows PE` (Windows Preinstallation Environment)

*   **Manual Disk Selection**: Automatic formatting or disk selection is not performed. The user will be able to manually select or configure disk partitions.

*   **EULA Acceptance**: The End-User License Agreement screen **will be displayed** for manual acceptance.

*   **Bypass System Requirements**: Disables checks for TPM 2.0, Secure Boot, and sufficient RAM.

### Pass 2: `Specialize`

*   **Script Extraction**: Unpacks all other scripts from the `autounattend.xml` file.

*   **Run `Specialize.ps1`**: This primary PowerShell script performs most system-wide configurations:

    *   **Disable AI Features**: Globally disables Cortana, Windows Copilot, and the AI screen analysis feature (Windows Recall).

    *   **Disable Automatic Driver Installation**: Prevents Windows Update from automatically downloading and installing device drivers.

    *   **Performance Optimization**:
        *   Disables Hibernation and Reserved Storage.
        *   Enables `CompactOS`.
        *   Globally disables background UWP apps (`LetAppsRunInBackground = 2`).
        *   Disables Power Throttling, Network Throttling, and sets System Responsiveness to 0.
        *   Disables Game DVR at the system level.

    *   **"Ultimate Performance" Power Plan Customization**:
        *   **CPU:** Sets Turbo Boost mode to `Aggressive` and disables Core Parking.
        *   **PCI Express:** Link State Power Management is set to `Off`.
        *   **USB:** Selective Suspend is `Disabled`.
        *   **Wi-Fi:** Adapter is set to `Maximum Performance`.

    *   **Disable Telemetry**: Stops and disables the `DiagTrack` service.

    *   **System Cleanup**: Removes OneDrive, the Microsoft Edge desktop shortcut, and uninstalls over 20 pre-installed UWP applications.
    
    *   **UI Customization**:
        *   Applies a custom taskbar layout.
        *   Cleans all default tiles from the Start Menu.
        *   Sets the system drive `C:` label to "Windows".

### Pass 3: `OobeSystem` (Out-of-Box Experience)

*   **Skip OOBE Screens**: Hides initial setup screens and the persistent prompt to sign in with a Microsoft Account.

### Post-Installation Tasks (FirstLogonCommands)

*   **Disable Mouse Acceleration** ("Enhance pointer precision").
*   **Disable Game Bar** at the user level.
*   **Configure File Explorer**: Sets "This PC" as the startup folder.
*   **JPEG Quality Settings**: Sets JPEG save quality to 100%.

## 📋 How to Use

1.  **Download `autounattend.xml`** from this repository.

2.  **Create a bootable USB drive** with an official Windows 10 or 11 image.

3.  **Place `autounattend.xml`** in the root of that USB drive.

4.  **Boot the computer** from the USB drive.

5.  The installation process will be automated. You will need to manually accept the EULA and select the disk for installation. The rest of the steps will be performed automatically.

> **‼️ NOTE:** This script **does not** automatically format the disk. You will be able to manually select and configure disk partitions during the installation process. Always back up all important data before installing an operating system.

### License / Disclaimer

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

### ⚠️ IMPORTANT:

* **Use at your own risk:** This answer file and instructions are provided "as is". The author is not liable for any system failures, data loss, or hardware damage.
* **External dependencies:** By installing libraries, you are using third-party software for which the author assumes no responsibility.
* **Code review:** It is highly recommended to study the code before execution and test it in a safe environment. Always create a backup of your important data before installing the operating system.

---

**Copyright © 2026 [Иван Бирючков / https://github.com/biryuchkov-ia]**

---