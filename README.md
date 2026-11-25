Плагин для Purpur / Paper 1.21.4, который делает две вещи:

Кастомные предметы — свои названия, описание, модели, характеристики, еда, зачарования и действия по правому клику. Всё описывается обычными .yml файлами.
Ресурспак-сервер — вы кладёте модели/текстуры в папку плагина (или готовый .zip), плагин сам собирает из этого один ресурспак, раздаёт его по HTTP и отправляет игрокам при входе.
Готовый файл: dist/KosmosAdder-1.0.0.jar (уже собран, можно сразу ставить).

Установка
Скопируйте KosmosAdder-1.0.0.jar в папку plugins/ вашего сервера.
Перезапустите сервер. Появятся папки:
plugins/KosmosAdder/
  config.yml
  items/                 <- сюда ваши предметы (*.yml)
  resourcepack/          <- сюда модели, текстуры, готовые zip
  pack/KosmosAdder-Pack.zip
Откройте config.yml и проверьте resource-pack:
port — порт встроенного HTTP-сервера (по умолчанию 8080, должен быть открыт наружу);
public-url — адрес, который увидят клиенты. Если сервер за панелью/прокси/докером, пропишите его вручную, например http://123.45.67.89:8080/pack.zip. Пустое значение = плагин попробует определить IP сам.
/kosmos reload — применить изменения без перезапуска.
Важно: клиент должен иметь возможность скачать пак по указанному адресу. Если играете через прокси (BungeeCord/Velocity) — адрес всё равно должен вести на тот сервер, где стоит плагин.

Быстрый старт: свой предмет за 3 шага
Положите текстуру 16×16: plugins/KosmosAdder/resourcepack/assets/kosmos/textures/item/my_item.png
Создайте plugins/KosmosAdder/items/my_item.yml:
id: my_item
material: PAPER
name: "<gradient:#7ef9ff:#4b6cff><bold>Мой предмет</bold></gradient>"
lore:
  - "<gray>Сделан на сервере</gray>"
/kosmos reload → /kosmos give my_item
Плагин сам создаст модель models/item/my_item.json и item-модель items/my_item.json — писать json руками не нужно.

Своя 3D-модель (Blockbench)
В Blockbench: File → Export → Java Block/Item Model.
Модель положите в resourcepack/assets/kosmos/models/item/<имя>.json, текстуру — в resourcepack/assets/kosmos/textures/item/<имя>.png.
В items/<id>.yml укажите:
model: "kosmos:item/<имя>"
/kosmos reload
Если файл модели уже существует, плагин не перезаписывает его — ваша модель в приоритете.

Готовый пак целиком
Можно просто закинуть готовый ресурспак архивом:

plugins/KosmosAdder/resourcepack/MyPack.zip
или plugins/KosmosAdder/resourcepack.zip
Плагин распакует архив, объединит с остальными файлами и добавит сгенерированные модели. При конфликте побеждает файл, лежащий в папке resourcepack/, а не в архиве.

Предметы (items/*.yml)
Файлы, имя которых начинается с _, не загружаются (_template.yml — шпаргалка со всеми полями).

Поле	Описание
id	Уникальный id (по умолчанию — имя файла)
material	Базовый предмет: PAPER, DIAMOND_SWORD, LEATHER_CHESTPLATE и т.д.
name	Название, формат MiniMessage
lore	Список строк описания, MiniMessage
model	Модель: kosmos:item/ruby. Если не указано — kosmos:item/<id>
item-model	Своя item-модель (assets/<ns>/items/<key>.json), необязательно
custom-model-data	Старый способ выбора модели (число)
glint	true/false — свечение зачарования
unbreakable	Не ломается
max-stack-size	Размер стака 1..99
max-damage	Прочность инструмента/брони
rarity	COMMON, UNCOMMON, RARE, EPIC
color	"#ff0055" — цвет кожаной брони
enchantments	sharpness: 5, unbreaking: 3, ...
attributes	attack_damage: 8, movement_speed: 0.05, ...
food	nutrition, saturation, can-always-eat
cooldown	Кулдаун действий в секундах
actions	Действия по правому клику
Пример с едой:

id: star_berry
material: SWEET_BERRIES
name: "<gradient:#e4a2ff:#7a37b2><bold>Звёздная ягода</bold></gradient>"
model: "kosmos:item/star_berry"
food:
  nutrition: 6
  saturation: 8.0
  can-always-eat: true
Действия по правому клику
cooldown: 3
actions:
  - "[message]<green>Предмет использован!</green>"
  - "[actionbar]<gray>+1 к настроению</gray>"
  - "[broadcast]<gold>%player% использовал %item%</gold>"
  - "[console]give %player% diamond 1"
  - "[player]spawn"
  - "[sound]block.amethyst_block.chime 1 1.4"
  - "[close]"
Плейсхолдеры: %player%, %item%, %world%.

Команды
Команда	Что делает
/kosmos help	Справка
/kosmos list [стр.]	Список предметов (клик = выдать себе)
/kosmos give <id> [кол-во]	Выдать себе
/kosmos give <игрок> <id> [кол-во]	Выдать игроку
/kosmos info <id>	Подробности предмета
/kosmos reload	Перечитать конфиг + предметы, пересобрать пак, разослать игрокам
/kosmos pack	URL, SHA-1, размер, состояние сервера пака
/kosmos pack build	Пересобрать и разослать пак
/kosmos pack resend [игрок]	Отправить пак заново
Алиасы команды: /ka, /kosmosadder.

Права: kosmosadder.admin (всё) и kosmosadder.give (выдача), по умолчанию op.

Как это работает
Предмет получает компонент minecraft:item_model (kosmos:ruby), который ссылается на assets/kosmos/items/ruby.json — новая система item-моделей 1.21.4.
Если модели нет, генерируется обычная 2D-модель item/generated на текстуру assets/kosmos/textures/item/<имя>.png.
При старте и по /kosmos reload папка resourcepack/ (+ архивы) упаковывается в один zip, для него считается SHA-1, и он раздаётся встроенным HTTP-сервером (/pack.zip).
При входе игрока плагин вызывает Player#setResourcePack(...) с URL и SHA-1.
pack_format для 1.21.4 — 46 (плагин создаёт pack.mcmeta сам).
Сборка из исходников
Нужна Java 21+.

Windows: двойной клик по build.bat (при необходимости сам скачает Maven в .tools/).
Linux/macOS: ./build.sh
Вручную: mvn clean package
Результат: target/KosmosAdder-1.0.0.jar (и копия в dist/).

Частые проблемы
Игрок видит «Не удалось загрузить ресурспак». Проверьте, что адрес из /kosmos pack открывается в браузере с другого компьютера: порт закрыт фаерволом, либо сервер за NAT/панелью. Пропишите resource-pack.public-url. Адрес должен быть доступен по HTTP(S) без авторизации.

Пак скачался, но предметы выглядят как обычная бумага. Текстура не найдена или лежит не по пути. В консоли после /kosmos reload есть подсказки: Предмет 'ruby': не найдена текстура assets/kosmos/textures/item/ruby.png. Проверьте регистр букв (всё в нижнем регистре) и что это именно .png.

Модель не применилась после правки. /kosmos reload пересобирает пак, но клиент кэширует по SHA-1. Если SHA-1 не изменился, значит файл не попал в пак — проверьте путь и повторите /kosmos pack resend.

Порт 8080 занят. Смените resource-pack.port. Если порт меняется, не забудьте обновить public-url.

Свой pack.mcmeta. Можно положить свой — плагин его не тронет, но при pack_format ≠ 46 напишет предупреждение.
