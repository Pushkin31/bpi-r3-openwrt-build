# Banana Pi BPI-R3 OpenWrt ImageBuilder Pipeline

Современный и невероятно быстрый репозиторий для автоматической сборки кастомной прошивки OpenWrt под **Banana Pi BPI-R3** с предустановленным **AmneziaVPN**.

## Что делает репозиторий

Вместо компиляции исходного кода OpenWrt с нуля (что занимает часы и часто приводит к ошибкам зависимостей ядра), этот пайплайн использует официальный **OpenWrt ImageBuilder**. 

- ⚡️ Сборка полностью готовой прошивки занимает **всего 2-4 минуты**.
- 📦 Автоматически скачивает прекомпилированные пакеты AmneziaWG напрямую из релизов [Slava-Shchipunov/awg-openwrt](https://github.com/Slava-Shchipunov/awg-openwrt).
- 🔄 Поддерживает динамическое переключение между старыми (24.10) и новыми (25.12+) форматами пакетов (`.ipk` и `.apk`).
- 🚀 Публикация готовых `.img.gz` образов и списка пакетов (`.manifest`) в GitHub Releases.

## Структура

- `.github/workflows/reusable-openwrt-build.yml` — ядро сборки на базе ImageBuilder.
- `.github/workflows/build.yml` — сборка артефактов без публикации (для тестов).
- `.github/workflows/release.yml` — сборка и автоматическое создание GitHub Release.
- `.config/config` — список кастомных пакетов, которые вы хотите добавить в прошивку.

## Как работает CI/CD

### Ручной запуск (Workflow Dispatch)
При запуске экшена вы можете указать две переменные:
1. **OpenWrt version to use**: Версия OpenWrt (по умолчанию `24.10.5`). Можно указать, например, `25.12.0`. Пайплайн сам подберет правильный архиватор и формат пакетов.
2. **Custom build number override (optional)**: Позволяет переопределить номер билда в названии релиза (например, вписать `4`, чтобы релиз назывался `#4.1`), игнорируя системный счетчик попыток GitHub.

### Результат сборки
В разделе Artifacts (или в Releases) появятся:
- `*bananapi_bpi-r3-sdcard.img.gz` — образ для записи на SD-карту.
- `*bananapi_bpi-r3.manifest` — текстовый файл с полным списком всех установленных в прошивку пакетов.
- `sha256sums` — контрольные суммы.

## Как менять состав пакетов

1. Откройте файл `.config/config`.
2. Добавьте нужные пакеты в формате `CONFIG_PACKAGE_<name>=y` или `=m`. 
   *(Примечание: скрипт автоматически отрезает префиксы и суффиксы, передавая в ImageBuilder чистые имена пакетов).*
3. Сделайте commit/push. 

Пакеты AmneziaWG зашиты в саму логику скачивания в `.github/workflows/reusable-openwrt-build.yml` и устанавливаются принудительно:
- `amneziawg-tools`
- `kmod-amneziawg`
- `luci-proto-amneziawg`

## Связанный проект (сервер)

Для серверной части VPN можно использовать ваш форк:
- [amneziawg-docker-server](https://github.com/Pushkin31/amneziawg-docker-server)

## Быстрый старт

1. Залейте этот код в свой репозиторий на GitHub.
2. В настройках репозитория: `Settings -> Actions -> General -> Workflow permissions -> Read and write permissions` (для создания релизов).
3. Перейдите во вкладку **Actions**.
4. Запустите:
   - **Build OpenWrt Firmware (Build-Only)** для тестовой сборки (сохранится в артефактах).
   - **Build OpenWrt Firmware (Release)** для публикации на главной странице репозитория.

## Установка и прошивка (Banana Pi BPI-R3)

Плата BPI-R3 имеет переключатели (джамперы) для выбора устройства загрузки. 
- **1** = On/Вверх
- **0** = Off/Вниз
- **X** = Не имеет значения (вторичный селектор)

| Режим загрузки | A | B | C (NAND/NOR) | D (SD/eMMC) |
| :--- | :--- | :--- | :--- | :--- |
| **SD Карта** | 1 | 1 | X | 1 |
| **eMMC** | 0 | 1 | X | 0 |
| **NAND** | 1 | 0 | 1 | X |
| **NOR** | 0 | 0 | 0 | X |

### 1. Загрузка с SD-карты
1. Скачайте файл `*bananapi_bpi-r3-sdcard.img.gz` из релизов.
2. Запишите его на SD-карту с помощью **BalenaEtcher**, **Rufus** или через терминал (`dd`).
3. Установите переключатели в режим **SD Карта** (A=1, B=1, C=1, D=1).
4. Вставьте карту и включите роутер.

### 2. Установка во внутреннюю память (eMMC / NAND / NOR)
Загрузившись с SD-карты, вы можете перенести систему во внутреннюю память прямо из терминала OpenWrt (через SSH):

**Перенос в NAND:**
```bash
fw_setenv bootcmd "env default bootcmd ; saveenv ; run ubi_init ; bootmenu 0"
reboot
```
*(После перезагрузки установите джамперы в режим NAND: A=1, B=0, C=1, D=0)*

**Перенос в NOR:**
```bash
fw_setenv bootcmd "env default bootcmd ; saveenv ; run nor_init ; bootmenu 0"
reboot
```
*(Джамперы в режим NOR: A=0, B=0, C=0, D=0)*

**Перенос в eMMC (с NAND):**
```bash
fw_setenv bootcmd "env default bootcmd ; saveenv ; run emmc_init ; bootmenu 0"
reboot
```
*(Джамперы в режим eMMC: A=0, B=1, C=1, D=0)*

## Лицензия

См. `LICENSE.md`.
