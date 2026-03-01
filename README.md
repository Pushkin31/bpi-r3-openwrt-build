# Banana Pi BPI-R3 OpenWrt Build Pipeline

Минимальный репозиторий для автоматической сборки прошивки OpenWrt под **Banana Pi BPI-R3**.

## Что делает репозиторий

- Сборка firmware через GitHub Actions
- Smoke-тесты артефакта (gzip, размер, checksum)
- Публикация GitHub Release (в отдельном workflow)
- Проверка, что все `CONFIG_PACKAGE_*=y` из `.config/config` реально доступны для выбранного target/feeds

## Структура

- `.github/workflows/reusable-openwrt-build.yml` — общий reusable pipeline
- `.github/workflows/build.yml` — build-only (schedule/manual)
- `.github/workflows/release.yml` — build + test + release (push/manual)
- `.config/config` — целевой target и состав пакетов
- `feeds.conf`, `feeds.conf.d/*.conf` — фиды OpenWrt/дополнительные фиды

## Как работает CI/CD

### Build-only workflow
Файл: `.github/workflows/build.yml`

Триггеры:
- `schedule` (еженедельно)
- `workflow_dispatch` (ручной запуск)

Результат:
- собирает прошивку
- прогоняет smoke-тесты
- загружает артефакт `openwrt-firmware-bpi-r3`
- **не создаёт Release**

### Release workflow
Файл: `.github/workflows/release.yml`

Триггеры:
- `push` в `main`
- `workflow_dispatch` (ручной запуск)

Результат:
- собирает прошивку
- прогоняет smoke-тесты
- создаёт GitHub Release и прикладывает:
  - `*bananapi_bpi-r3-squashfs-sdcard.img.gz`
  - `sha256sums`

## Ручной запуск с выбором версии OpenWrt

Оба workflow поддерживают input `openwrt_ref`:
- tag (например `v24.10.0`)
- branch (например `main`)
- commit SHA

Если не указан, используется `v24.10.0`.

Есть ранняя валидация `openwrt_ref`: если ref не существует или формат некорректный, pipeline падает сразу на этапе проверки.

## Как менять состав пакетов

1. Отредактируйте `.config/config`
2. Добавьте/удалите строки вида:
   - `CONFIG_PACKAGE_<name>=y`
3. Сделайте commit/push

Pipeline сравнивает:
- запрошенные пакеты из вашего `.config/config`
- фактически разрешённые пакеты после `make defconfig`

Если часть пакетов недоступна для target/feeds — сборка завершится ошибкой со списком missing packages.

## Как менять фиды

- Изменяйте `feeds.conf`
- Добавляйте/редактируйте `feeds.conf.d/*.conf`

Они копируются в OpenWrt перед `./scripts/feeds update -a` и `./scripts/feeds install -a`.

Текущий feed Amnezia для OpenWrt:
- `https://github.com/amnezia-vpn/amnezia-openwrt-packages.git`

## Связанный проект (сервер)

Для серверной части можно использовать ваш форк:
- `https://github.com/Pushkin31/amneziawg-docker-server`

## Быстрый старт

1. Создайте пустой репозиторий на GitHub
2. Залейте этот набор файлов
3. В GitHub включите:
   - `Settings -> Actions -> General -> Workflow permissions -> Read and write permissions`
4. Запустите:
   - `Build OpenWrt Firmware (Build-Only)` для тестовой сборки
   - `Build OpenWrt Firmware (Release)` для публикации релиза

## Артефакты

Артефакт `openwrt-firmware-bpi-r3` содержит:
- образ прошивки `.img.gz`
- `sha256sums`
- `release_notes.txt`

## Лицензия

См. `LICENSE.md`.
