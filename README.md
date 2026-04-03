# Custom Items - понятная инструкция на русском

## Что это за мод

Этот мод добавляет **кастомные расходуемые предметы** для Minecraft Forge 1.7.10.

Проще говоря:

- ты описываешь предмет в файле `items.json`;
- сервер читает этот файл;
- по команде или при входе игроков мод раздаёт и синхронизирует эти предметы;
- предмет может:
  - иметь своё имя;
  - иметь свою иконку;
  - иметь свой размер стака;
  - проигрывать анимацию еды, питья или срабатывать мгновенно;
  - накладывать эффекты зелий.

Важно: это именно **расходники с эффектами**, а не универсальная система для мечей, кирок, брони и 3D-моделей.

## Что мод умеет и чего не умеет

### Умеет

- создавать предметы через JSON без переписывания Java-кода;
- выдавать предметы через команду;
- перезагружать конфиг без перезапуска сервера;
- показывать разные иконки для разных предметов;
- синхронизировать список предметов с клиентом;
- использовать как стандартные текстуры Minecraft, так и свои.

### Не умеет

- добавлять новый тип предмета кроме расходуемого;
- лечить голод как обычная еда;
- задавать урон, прочность, броню, инструментальные свойства;
- читать PNG прямо из папки `config`;
- автоматически создавать текстуру по имени.

## Как мод работает простыми словами

Ниже самая важная идея.

В коде реально зарегистрирован **один базовый предмет**:

- он называется `configurable_consumable`;
- а все твои "разные" предметы отличаются не отдельными Java-классами, а значением `CustomItemId` внутри NBT.

То есть мод работает так:

1. Сервер запускается.
2. Мод читает файл `items.json`.
3. Из этого файла собирается список предметов.
4. Когда игрок заходит на сервер, сервер отправляет ему этот список.
5. Клиент рисует имя, иконку, подсказку и поведение предмета по этому описанию.

Поэтому вся настройка делается в одном месте: **в JSON-файле и в ресурсах с текстурами**.

## Где лежит главный файл с предметами

### В обычной сборке / на сервере

После первого запуска мода появится файл:

```text
.minecraft/config/customitems/items.json
```

Если это сервер, путь будет такого вида:

```text
server/config/customitems/items.json
```

### В dev-среде этого проекта

В этом проекте Gradle использует папку `eclipse` как `runDir`, поэтому сейчас файл находится тут:

```text
eclipse/config/customitems/items.json
```

## Что будет после первого запуска

Если файла ещё нет, мод создаёт пример:

```json
{
  "items": [
    {
      "id": "starter_tonic",
      "displayName": "Starter Tonic",
      "texture": "minecraft:potion_bottle_drinkable",
      "stackSize": 16,
      "useAnimation": "drink",
      "enabled": true,
      "effects": [
        {
          "type": "regeneration",
          "level": 2,
          "durationSeconds": 15
        },
        {
          "type": "speed",
          "level": 1,
          "durationSeconds": 30
        }
      ]
    }
  ]
}
```

Это хороший шаблон: можно просто копировать его и менять значения.

## Самый быстрый способ создать свой предмет

1. Открой `items.json`.
2. Найди массив `"items"`.
3. Скопируй существующий объект предмета.
4. Измени `id`, `displayName`, `texture` и `effects`.
5. Сохрани файл.
6. В игре выполни команду:

```text
/customitems reload
```

7. Проверь список предметов:

```text
/customitems list
```

8. Выдай предмет себе:

```text
/customitems give <твой_ник> <itemId> [количество]
```

Пример:

```text
/customitems give Admin starter_tonic 3
```

## Минимальный рабочий шаблон

Если хочешь начать с самого простого варианта, используй вот это:

```json
{
  "items": [
    {
      "id": "my_first_item",
      "displayName": "Мой первый предмет",
      "texture": "minecraft:apple",
      "stackSize": 16,
      "useAnimation": "drink",
      "enabled": true,
      "effects": [
        {
          "type": "speed",
          "level": 1,
          "durationSeconds": 30
        }
      ]
    }
  ]
}
```

## Полный разбор каждого поля

### `id`

Это **внутренний ID предмета**. Через него мод понимает, какой именно предмет ты имеешь в виду.

Пример:

```json
"id": "cherry_juice"
```

Что важно:

- должен быть уникальным;
- лучше писать только маленькими буквами;
- безопасный формат: буквы, цифры и `_`;
- мод сам приводит `id` к нижнему регистру и заменяет неподходящие символы на `_`;
- именно этот ID используется в команде `/customitems give`.

Если два предмета будут с одинаковым `id`, второй будет пропущен.

### `displayName`

Это имя, которое игрок увидит в игре.

Пример:

```json
"displayName": "Вишнёвый сок"
```

Если не указать `displayName`, мод сам сделает имя из `id`.

Пример:

- `starter_tonic` превратится в `Starter Tonic`.

### `texture`

Это ссылка на текстуру в формате:

```text
мод:путь
```

Примеры:

```json
"texture": "minecraft:apple"
```

```json
"texture": "minecraft:potion_bottle_drinkable"
```

```json
"texture": "customitems:drinks/cherry_juice"
```

Очень важно понять:

- это **не путь до файла на диске**;
- это **имя ресурса Minecraft**;
- PNG должен лежать в правильной папке `assets/.../textures/items/...`.

Подробно про это ниже в разделе про текстуры.

Если текстура не найдена, предмет обычно покажется с запасной иконкой `paper`.

### `stackSize`

Сколько штук помещается в одном стаке.

Пример:

```json
"stackSize": 16
```

Что важно:

- мод всё равно ограничивает значение от `1` до `64`;
- если написать `0`, предмет фактически станет `1`;
- если написать `999`, мод всё равно сделает `64`.

### `useAnimation`

Как предмет используется.

Допустимые значения:

- `"drink"` - как зелье;
- `"eat"` - как еда;
- `"instant"` - срабатывает сразу, без ожидания.

Примеры:

```json
"useAnimation": "drink"
```

```json
"useAnimation": "eat"
```

```json
"useAnimation": "instant"
```

Как это работает:

- `drink` - игрок зажимает ПКМ, идёт анимация питья, потом применяются эффекты;
- `eat` - то же самое, но с анимацией еды;
- `instant` - эффект применяется сразу при нажатии.

Если написать неверное значение, мод откатится к `"drink"`.

### `enabled`

Включён ли предмет.

Пример:

```json
"enabled": true
```

Или:

```json
"enabled": false
```

Если поставить `false`, предмет просто не загрузится.

### `effects`

Это список эффектов, которые накладываются при использовании.

Пример:

```json
"effects": [
  {
    "type": "speed",
    "level": 1,
    "durationSeconds": 30
  }
]
```

Важно:

- `effects` должен быть массивом;
- в предмете должен быть хотя бы один корректный эффект;
- если все эффекты неправильные, предмет будет пропущен.

## Разбор полей внутри `effects`

Каждый эффект выглядит так:

```json
{
  "type": "speed",
  "level": 1,
  "durationSeconds": 30
}
```

### `type`

Это ID эффекта.

Поддерживаются понятные алиасы. Например:

- `speed`
- `slowness`
- `haste`
- `mining_fatigue`
- `strength`
- `instant_health`
- `heal`
- `instant_damage`
- `harm`
- `jump_boost`
- `nausea`
- `regeneration`
- `resistance`
- `fire_resistance`
- `water_breathing`
- `invisibility`
- `blindness`
- `night_vision`
- `hunger`
- `weakness`
- `poison`
- `wither`
- `health_boost`
- `absorption`
- `saturation`

Мод также понимает часть коротких вариантов:

- `regen`
- `jump`
- `slow`

Пример:

```json
"type": "regeneration"
```

### `level`

Это уровень эффекта.

Можно писать:

```json
"level": 1
```

или:

```json
"level": "II"
```

Оба варианта работают.

Поддерживаются:

- обычные числа;
- римские числа от `I` до `X`.

Как понимать уровень:

- `1` = I;
- `2` = II;
- `3` = III.

Если уровень меньше `1`, эффект будет пропущен.

### `durationSeconds`

Длительность в секундах.

Пример:

```json
"durationSeconds": 45
```

Что важно:

- для обычных эффектов значение должно быть больше `0`;
- для мгновенных эффектов вроде `instant_health` и `instant_damage` ноль допустим.

## Готовые примеры предметов

### 1. Напиток на скорость

```json
{
  "id": "speed_drink",
  "displayName": "Напиток скорости",
  "texture": "minecraft:potion_bottle_drinkable",
  "stackSize": 16,
  "useAnimation": "drink",
  "enabled": true,
  "effects": [
    {
      "type": "speed",
      "level": 2,
      "durationSeconds": 30
    }
  ]
}
```

### 2. Еда с несколькими эффектами

```json
{
  "id": "battle_cookie",
  "displayName": "Боевой пряник",
  "texture": "minecraft:cookie",
  "stackSize": 32,
  "useAnimation": "eat",
  "enabled": true,
  "effects": [
    {
      "type": "strength",
      "level": 2,
      "durationSeconds": 20
    },
    {
      "type": "resistance",
      "level": 1,
      "durationSeconds": 20
    }
  ]
}
```

### 3. Мгновенное лечение

```json
{
  "id": "heal_injector",
  "displayName": "Инъектор лечения",
  "texture": "minecraft:ghast_tear",
  "stackSize": 8,
  "useAnimation": "instant",
  "enabled": true,
  "effects": [
    {
      "type": "instant_health",
      "level": 2,
      "durationSeconds": 0
    }
  ]
}
```

## Как добавить свой предмет в `items.json`

Если у тебя уже есть файл с несколькими предметами, добавляй новый объект **внутрь массива** `"items"`.

Пример:

```json
{
  "items": [
    {
      "id": "starter_tonic",
      "displayName": "Starter Tonic",
      "texture": "minecraft:potion_bottle_drinkable",
      "stackSize": 16,
      "useAnimation": "drink",
      "enabled": true,
      "effects": [
        {
          "type": "regeneration",
          "level": 2,
          "durationSeconds": 15
        }
      ]
    },
    {
      "id": "battle_cookie",
      "displayName": "Боевой пряник",
      "texture": "minecraft:cookie",
      "stackSize": 32,
      "useAnimation": "eat",
      "enabled": true,
      "effects": [
        {
          "type": "strength",
          "level": 2,
          "durationSeconds": 20
        }
      ]
    }
  ]
}
```

Главное:

- между объектами должна быть запятая;
- после последнего объекта лишнюю запятую ставить нельзя;
- JSON очень чувствителен к скобкам и запятым.

## Как добавить текстуру предмета

Это самая частая точка путаницы, поэтому ниже максимально просто.

### Самая важная мысль

Поле `"texture"` хранит **не путь до PNG**, а **имя ресурса Minecraft**.

Когда ты пишешь:

```json
"texture": "customitems:drinks/cherry_juice"
```

Minecraft ищет файл здесь:

```text
assets/customitems/textures/items/drinks/cherry_juice.png
```

То есть:

- `customitems` - это namespace, обычно modid;
- `drinks/cherry_juice` - это путь внутри `textures/items`;
- `.png` дописывается автоматически.

### Вариант 1. Использовать уже существующую текстуру Minecraft

Это самый простой вариант.

Примеры:

```json
"texture": "minecraft:apple"
```

```json
"texture": "minecraft:cookie"
```

```json
"texture": "minecraft:potion_bottle_drinkable"
```

Плюс такого способа:

- ничего дополнительно создавать не нужно.

### Вариант 2. Добавить свою текстуру через resource pack

Это лучший вариант, если ты хочешь менять картинки без пересборки мода.

#### Шаг 1. Создай папку resource pack

Например:

```text
MyCustomItemsPack
```

#### Шаг 2. Внутри создай `pack.mcmeta`

Для Minecraft 1.7.10:

```json
{
  "pack": {
    "pack_format": 1,
    "description": "Textures for Custom Items"
  }
}
```

#### Шаг 3. Создай правильные папки

Если хочешь использовать строку:

```json
"texture": "customitems:drinks/cherry_juice"
```

то структура должна быть такой:

```text
MyCustomItemsPack/
  pack.mcmeta
  assets/
    customitems/
      textures/
        items/
          drinks/
            cherry_juice.png
```

#### Шаг 4. Положи PNG

Файл должен называться:

```text
cherry_juice.png
```

Лучше всего делать обычную квадратную иконку:

- `16x16`;
- `32x32`;
- `64x64`.

#### Шаг 5. Включи resource pack в игре

Папку или zip-пак положи в:

```text
.minecraft/resourcepacks
```

Потом включи его в настройках ресурсов Minecraft.

#### Шаг 6. Укажи текстуру в `items.json`

```json
"texture": "customitems:drinks/cherry_juice"
```

#### Шаг 7. Перезагрузи ресурсы

Самый надёжный способ:

- нажать `F3 + T`;
- или перевключить resource pack;
- или изменить `items.json` и выполнить `/customitems reload`, чтобы мод заново синхронизировал описания.

### Вариант 3. Вшить текстуру прямо в мод

Если ты работаешь именно с исходниками этого проекта и собираешь свой jar, PNG можно положить прямо в ресурсы мода.

Нужный путь:

```text
src/main/resources/assets/customitems/textures/items/
```

Пример:

```text
src/main/resources/assets/customitems/textures/items/drinks/cherry_juice.png
```

Тогда в `items.json` будет то же самое:

```json
"texture": "customitems:drinks/cherry_juice"
```

После этого мод нужно пересобрать.

## Пошаговый пример: делаем предмет со своей текстурой

Допустим, ты хочешь предмет **"Вишнёвый сок"**.

### 1. Кладёшь текстуру сюда

```text
assets/customitems/textures/items/drinks/cherry_juice.png
```

### 2. В `items.json` добавляешь предмет

```json
{
  "id": "cherry_juice",
  "displayName": "Вишнёвый сок",
  "texture": "customitems:drinks/cherry_juice",
  "stackSize": 16,
  "useAnimation": "drink",
  "enabled": true,
  "effects": [
    {
      "type": "speed",
      "level": 1,
      "durationSeconds": 20
    },
    {
      "type": "regeneration",
      "level": 1,
      "durationSeconds": 10
    }
  ]
}
```

### 3. Перезагружаешь мод

```text
/customitems reload
```

### 4. Выдаёшь предмет

```text
/customitems give Admin cherry_juice 1
```

Если всё сделано правильно, ты увидишь новый предмет с нужной иконкой.

## Команды мода

### `/customitems list`

Показывает список ID всех загруженных предметов.

Пример:

```text
/customitems list
```

### `/customitems give <player> <itemId> [amount]`

Выдаёт предмет игроку.

Пример:

```text
/customitems give Admin battle_cookie 5
```

Если количество больше размера стака, мод сам разобьёт выдачу на несколько стаков.

### `/customitems reload`

Перечитывает `items.json` и заново отправляет список предметов всем игрокам.

Пример:

```text
/customitems reload
```

Важно:

- для выполнения команд нужны права уровня OP 2;
- после редактирования `items.json` обычно именно эту команду и нужно выполнять.

## Что увидит игрок в игре

У каждого предмета:

- будет своё имя из `displayName`;
- будет иконка из `texture`;
- в подсказке будет показываться:
  - внутренний ID;
  - размер стака;
  - список эффектов и их длительность.

Мод специально отключает обычный зачарованный блеск у этих предметов.

## Почему предмет может не появиться или работать не так

Ниже самые частые причины.

### 1. `enabled: false`

Такой предмет мод просто не загружает.

### 2. Два одинаковых `id`

Если ID повторяется, дубликат пропускается.

### 3. Нет ни одного корректно оформленного эффекта

Если список `effects` пустой или все эффекты ошибочные по формату, предмет не загрузится.

Примеры таких ошибок:

- пустой `type`;
- `level` меньше `1`;
- `durationSeconds` меньше или равен `0` у обычного эффекта.

### 4. Неправильный JSON

Лишняя запятая, забытая скобка или кавычка ломают загрузку файла.

### 5. Неправильный `type`

Если эффект не распознан Minecraft, предмет всё равно может загрузиться, но сам эффект не будет применяться.

### 6. Неправильная текстура

Если строка `"texture"` указывает на несуществующий ресурс, предмет появится, но иконка будет неправильной или запасной.

### 7. Не выполнен reload

После изменения `items.json` обычно нужно:

```text
/customitems reload
```

## Если текстура не отображается

Проверь по этому списку:

1. Совпадает ли namespace.

Если у тебя:

```json
"texture": "customitems:drinks/cherry_juice"
```

то папка должна начинаться с:

```text
assets/customitems/
```

2. Совпадает ли путь.

Если в JSON написано:

```json
"texture": "customitems:drinks/cherry_juice"
```

то PNG должен быть здесь:

```text
assets/customitems/textures/items/drinks/cherry_juice.png
```

3. Совпадает ли имя файла.

`Cherry_Juice.png` и `cherry_juice.png` лучше не путать.

4. Перезагружены ли ресурсы.

Используй:

```text
F3 + T
```

5. Включён ли resource pack.

Если PNG лежит в resource pack, но сам pack выключен, текстура не появится.

## Если хочешь понять, "откуда что берётся"

Ниже короткая карта мода человеческим языком.

### Откуда берутся предметы

Из файла:

```text
config/customitems/items.json
```

Этим занимается менеджер конфига.

### Откуда берётся имя предмета

Из поля:

```json
"displayName"
```

Если его нет, имя собирается из `id`.

### Откуда берётся иконка

Из поля:

```json
"texture"
```

Но сама картинка должна существовать в ресурсах Minecraft.

### Откуда берётся поведение при использовании

Из поля:

```json
"useAnimation"
```

И из списка:

```json
"effects"
```

### Откуда берётся сам выданный предмет

Команда `/customitems give` создаёт обычный `ItemStack` базового предмета и записывает в NBT:

```text
CustomItemId
```

По этому ID мод понимает, какой именно это кастомный предмет.

## Если тебе нужно быстрое правило в одну строку

Запомни так:

> `items.json` отвечает за логику предмета, а `assets/.../textures/items/...png` отвечает за картинку.

## Рекомендуемый порядок работы

Вот самый удобный порядок, если ты добавляешь новый предмет:

1. Сначала придумай `id`.
2. Потом подготовь PNG.
3. Потом пропиши `"texture": "modid:путь"`.
4. Потом добавь эффекты в `items.json`.
5. Сохрани файл.
6. Выполни `/customitems reload`.
7. Выдай предмет через `/customitems give`.

## Полезный готовый шаблон для копирования

```json
{
  "id": "new_item_id",
  "displayName": "Название предмета",
  "texture": "customitems:new_item_id",
  "stackSize": 16,
  "useAnimation": "drink",
  "enabled": true,
  "effects": [
    {
      "type": "speed",
      "level": 1,
      "durationSeconds": 30
    }
  ]
}
```

Если используешь этот шаблон, не забудь положить PNG сюда:

```text
assets/customitems/textures/items/new_item_id.png
```

## Для тех, кто хочет знать, какие части кода за что отвечают

Это уже не обязательно для использования, но полезно для ориентира:

- `src/main/java/ru/customitems/config/ModConfigManager.java`
  - создаёт папку конфига;
  - создаёт пример `items.json`;
  - читает и валидирует предметы.
- `src/main/java/ru/customitems/config/CustomItemRegistry.java`
  - хранит список загруженных предметов;
  - создаёт `ItemStack` по `id`.
- `src/main/java/ru/customitems/item/ConfigurableConsumableItem.java`
  - задаёт имя, иконку, стаки, подсказку и поведение при использовании.
- `src/main/java/ru/customitems/config/PotionEffectResolver.java`
  - переводит `speed`, `regen`, `instant_health` и другие строки в реальные эффекты Minecraft.
- `src/main/java/ru/customitems/command/CommandCustomItems.java`
  - реализует команды `list`, `give`, `reload`.
- `src/main/java/ru/customitems/sync/ServerSyncManager.java`
  - отправляет список предметов игрокам.

## Короткий итог

Если совсем коротко:

1. Предметы описываются в `items.json`.
2. Иконки берутся из обычных ресурсов Minecraft.
3. Свой PNG нужно класть в `assets/<namespace>/textures/items/...`.
4. После изменений нужно делать `/customitems reload`.
5. Для проверки используй `/customitems list` и `/customitems give`.
