<div align="center">

<img src="assets/banner.svg" alt="Симулятор iOS 26.5 на Windows" width="100%">

[English](README.md) | **Русский**

[![Инструкция PDF](https://img.shields.io/badge/инструкция-PDF%20·%2024%20стр.-0969da?style=flat-square)](docs/iphone-simulator-on-windows_ru.pdf)
![Host](https://img.shields.io/badge/host-Windows%2010%2F11-1f2328?style=flat-square&logo=windows)
![VMware](https://img.shields.io/badge/VMware%20Workstation-26H1-1f2328?style=flat-square&logo=vmware)
![macOS](https://img.shields.io/badge/macOS-Tahoe%2026-1f2328?style=flat-square&logo=apple)
![Xcode](https://img.shields.io/badge/Xcode-26.6-1f2328?style=flat-square&logo=xcode)
![iOS](https://img.shields.io/badge/iOS%20Simulator-26.5-1f2328?style=flat-square&logo=ios)
[![License](https://img.shields.io/badge/license-GPL--3.0-1f2328?style=flat-square)](LICENSE)

Пошаговая инструкция: как развернуть **macOS Tahoe** в **VMware Workstation**, установить **Xcode 26.6** и запустить **симулятор iOS 26.5**, самую свежую версию iOS, доступную на компьютере с процессором Intel.

[**Скачать инструкцию (PDF)**](docs/iphone-simulator-on-windows_ru.pdf) · [English PDF](docs/iphone-simulator-on-windows_en.pdf)

</div>

---

## Содержание

- [Почему iOS 26.5, а не iOS 27](#почему-ios-265-а-не-ios-27)
- [Что понадобится](#что-понадобится)
- [Версии программ](#версии-программ)
- [Весь процесс](#весь-процесс)
- [Шпаргалка](#шпаргалка)
- [Решение проблем](#решение-проблем)
- [Как вернуть Windows в исходное состояние](#как-вернуть-windows-в-исходное-состояние)
- [Ограничения](#ограничения)
- [Юридическое предупреждение](#юридическое-предупреждение)
- [Источники](#источники)

## Почему iOS 26.5, а не iOS 27

Xcode 27 (вышел 14 сентября 2026) работает только на Mac с чипами Apple Silicon, а **macOS 26 Tahoe стала последней версией macOS для процессоров Intel**. На обычном ПК можно запустить только Intel-версию macOS, поэтому потолок такой:

```
macOS Tahoe 26  →  Xcode 26.6  →  симулятор iOS 26.5
```

Это последняя связка, которая вообще доступна на x86-компьютерах.

| План | macOS в виртуалке | Xcode | Симулятор iOS | Стабильность в VM |
| :-- | :-- | :-- | :-- | :-- |
| **A** (основной) | Tahoe 26.2 и новее | 26.6 | **26.5** | Работает, поддержка ограниченная |
| **B** (запасной) | Sequoia 15.6 и новее | 26.3 | 26.2 | Стабильнее, меньше проблем с мышью |

## Что понадобится

| | Минимум | Рекомендация |
| :-- | :-- | :-- |
| **Процессор** | Intel с AVX2 (Core 4-го поколения, Haswell, и новее) | Intel лучше всего подходит для macOS в VMware |
| **Виртуализация** | Intel VT-x включена в BIOS/UEFI | |
| **Память** | 16 ГБ | 32 ГБ: 16 ГБ виртуалке, 16 ГБ остаётся Windows |
| **Диск** | 100 ГБ свободного места | 150 ГБ на SSD (на HDD всё очень медленно) |
| **Система** | Windows 10 или 11, 64-бит | Нужны права администратора |
| **Интернет** | Стабильный, около 30 ГБ трафика | macOS скачивается с серверов Apple во время установки |
| **Apple Account** | Бесплатный | Нужен только для скачивания Xcode |

> [!NOTE]
> Весь процесс занимает **2-4 часа**, большая часть уходит на скачивание и установку macOS.

## Версии программ

| Программа | Версия | Откуда |
| :-- | :-- | :-- |
| VMware Workstation Pro | 26H1 (или 26H1u1) | Портал Broadcom, бесплатно |
| Unlocker (BDisp) | 3.1.4 | [github.com/BDisp/unlocker](https://github.com/BDisp/unlocker/releases) |
| recoveryOS (DrDonk) | последняя | [github.com/DrDonk/recoveryOS](https://github.com/DrDonk/recoveryOS/releases) |
| QEMU (нужен только `qemu-img`) | последняя | `winget install --id SoftwareFreedomConservancy.QEMU` |
| macOS Tahoe | 26.x | Скачивается с серверов Apple при установке |
| Xcode | 26.6, сборка **Universal** | [developer.apple.com/download/all](https://developer.apple.com/download/all) |
| Симулятор iOS | 26.5 | Скачивается из Xcode |

## Весь процесс

| # | Этап | Что делаем | Время |
| :-: | :-- | :-- | :-- |
| 1 | **Подготовка Windows** | Проверка VT-x, отключение Hyper-V и целостности памяти | ~15 мин + перезагрузка |
| 2 | **VMware Workstation** | Скачивание с портала Broadcom и установка | ~15 мин |
| 3 | **Unlocker** | Патч, который добавляет macOS в список гостевых систем | ~5 мин |
| 4 | **Образ восстановления** | Скачивание recoveryOS Tahoe с серверов Apple, конвертация в VMDK | ~15 мин |
| 5 | **Виртуальная машина** | Создание VM, подключение диска восстановления | ~10 мин |
| 6 | **Установка macOS** | Разметка диска, установка по сети, первичная настройка | 60-90 мин |
| 7 | **После установки** | VMware Tools, ускорение, общие папки | ~15 мин |
| 8 | **Xcode и симулятор** | Xcode 26.6, платформа iOS 26.5, запуск iPhone | 45-60 мин |

В PDF каждый этап расписан по шагам, с макетами всех окон и отмеченными полями.

## Шпаргалка

<details>
<summary><b>1. Отключить Hyper-V</b> (PowerShell от имени администратора)</summary>

```powershell
dism.exe /Online /Disable-Feature:Microsoft-Hyper-V-All /NoRestart
dism.exe /Online /Disable-Feature:VirtualMachinePlatform /NoRestart
dism.exe /Online /Disable-Feature:HypervisorPlatform /NoRestart
bcdedit /set hypervisorlaunchtype off
```

Также выключите **Безопасность Windows → Безопасность устройства → Изоляция ядра → Целостность памяти** и перезагрузитесь.

> [!WARNING]
> После этого перестанут запускаться WSL 2, Docker Desktop, Песочница Windows и эмуляторы Android на базе Hyper-V. Как всё вернуть, см. [раздел ниже](#как-вернуть-windows-в-исходное-состояние).

</details>

<details>
<summary><b>3. Применить Unlocker</b> (cmd от имени администратора, VMware закрыта)</summary>

```bat
tasklist | findstr /I "vmware vmx"
cd /d "%USERPROFILE%\Downloads\unlocker-3.1.4"
win-install.cmd
```

Запускайте `win-install.cmd` повторно после **каждого** обновления VMware: обновление возвращает оригинальные файлы.

</details>

<details>
<summary><b>4. Получить образ восстановления</b> (PowerShell)</summary>

```powershell
# Установить qemu-img
winget install --id SoftwareFreedomConservancy.QEMU

# Ручное скачивание recovery Tahoe и конвертация в VMDK
.\macrecovery.exe -action download -board-id Mac-E1008331FDC96864 -mlb 00000000000000000 `
  -basename tahoe -outdir . -board-db .\boards.json
qemu-img convert -p -O vmdk -o compat6 tahoe.dmg tahoe.vmdk
```

</details>

<details>
<summary><b>5. Параметры виртуальной машины</b></summary>

| Параметр | Значение |
| :-- | :-- |
| Конфигурация | Custom (advanced) |
| Гостевая система | Apple Mac OS X → macOS 26 |
| Процессор | **1** процессор × **4** ядра (только чётные значения) |
| Память | 16384 МБ |
| Диск | SATA, 150 ГБ, одним файлом, без предварительного выделения |
| Второй диск | Существующий `tahoe.vmdk`, SATA, Keep Existing Format |

Ключевые строки файла `.vmx`:

```ini
guestOS = "darwin25-64"        # Sequoia (план B): "darwin24-64"
memsize = "16384"
numvcpus = "4"
cpuid.coresPerSocket = "4"
```

</details>

<details>
<summary><b>8. Xcode и симулятор</b> (Терминал macOS)</summary>

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
sudo xcodebuild -license accept
xcodebuild -runFirstLaunch

# Платформа iOS
xcodebuild -downloadPlatform iOS
xcrun simctl list runtimes

# Запустить iPhone и открыть сайт
xcrun simctl boot "iPhone 17 Pro"
open -a Simulator
xcrun simctl openurl booted "https://example.com"
```

> [!IMPORTANT]
> Скачивайте сборку Xcode **Universal**. Файл с пометкой «Apple silicon» в виртуалке не запустится.

</details>

## Решение проблем

| Проблема | Решение |
| :-- | :-- |
| VT-x недоступна, машина не включается | Включите Intel VT-x в BIOS, отключите Hyper-V (этап 1) |
| Всё очень медленно, в `vmware.log` режим монитора ULM | VMware работает поверх Hyper-V. Повторите этап 1, проверьте `msinfo32` |
| Нет логотипа Apple, перезагрузка по кругу, «The CPU has been disabled» | Повторите Unlocker, сверьте `guestOS` с образом, выставьте 1 процессор × 4 ядра |
| Tools установлены, но разрешение не меняется и мышь тормозит | Нажмите **Разрешить** в «Конфиденциальность и безопасность», перезагрузите macOS |
| «Xcode не поддерживается на этом Mac» | Скачана сборка для Apple silicon. Нужна **Universal** |
| Платформа iOS не скачивается из Xcode | `xcodebuild -downloadPlatform iOS` или ручная установка `.dmg` |
| Unlocker не работает на вашей версии VMware | Попробуйте [OC4VM](https://github.com/DrDonk/OC4VM): готовые шаблоны машин с OpenCore |

Полная таблица на страницах 22-23 PDF, там же схема перехода на **план B (Sequoia)**.

## Как вернуть Windows в исходное состояние

```powershell
bcdedit /set hypervisorlaunchtype auto
dism.exe /Online /Enable-Feature:VirtualMachinePlatform /All /NoRestart
dism.exe /Online /Enable-Feature:HypervisorPlatform /All /NoRestart
dism.exe /Online /Enable-Feature:Microsoft-Hyper-V-All /All /NoRestart   # только Pro/Enterprise
```

После этого перезагрузитесь и снова включите «Целостность памяти» в Безопасности Windows.

## Ограничения

- **Apple Account внутри macOS**: iCloud, App Store и вход в систему не работают. Вход на developer.apple.com через браузер работает, и этого достаточно для Xcode.
- **Какие приложения запускаются**: только сборки для симулятора (ваш проект в Xcode или `.app` для симулятора). Приложения из App Store и `.ipa` для реальных iPhone не запустятся.
- **Скорость**: видеоускорения нет, графику рисует процессор. Для проверки сайтов и отладки своего приложения этого хватает.
- **Liquid Glass**: эффекты прозрачности Tahoe в виртуалке не отображаются. На симулятор это не влияет.

## Юридическое предупреждение

> [!CAUTION]
> Лицензионное соглашение macOS разрешает устанавливать систему только на компьютеры Apple. Запуск macOS в виртуальной машине на обычном ПК это соглашение нарушает. Используйте инструкцию для личного ознакомления и тестирования на свой страх и риск. Для коммерческой разработки законный путь: Mac или облачный Mac (MacinCloud, MacStadium, AWS EC2 Mac).

## Источники

1. [Apple: Xcode system requirements](https://developer.apple.com/xcode/system-requirements/)
2. [Apple: Downloading and installing additional Xcode components](https://developer.apple.com/documentation/xcode/downloading-and-installing-additional-xcode-components)
3. [История версий Xcode](https://mungomash.com/software/xcode/versions)
4. [Broadcom KB 368734: Download Desktop Hypervisor products](https://knowledge.broadcom.com/external/article/368734)
5. [macOS on VMware Workstation 26.x](https://github.com/aesiddiqui/macos-on-vmware-workstation)
6. [BDisp/unlocker](https://github.com/BDisp/unlocker)
7. [DrDonk/recoveryOS](https://github.com/DrDonk/recoveryOS)
8. [DrDonk/OC4VM wiki](https://github.com/DrDonk/OC4VM/wiki)

## Лицензия

[GPL-3.0](LICENSE)
