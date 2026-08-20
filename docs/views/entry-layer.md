# Слой входа: схема

Две картинки. Первая отвечает на вопрос задания «клиент → шлюз / BFF → сервисы» на уровне магазина. Вторая делает зум внутрь и показывает то, что на общей схеме теряется: у сервиса уведомлений два входа, и только один из них проходит через шлюз.

Решения, которые здесь нарисованы, разобраны в [`09-entry-layer.md`](../09-entry-layer.md) и записаны в [ADR-0003](../adr/0003-entry-layer-gateway-and-bff.md).

## Клиент → шлюз / BFF → сервисы

```mermaid
graph TB
    W["Веб"]
    M["Мобильное приложение"]
    A["Админка"]

    W --> BW["Web BFF"]
    M --> BM["Mobile BFF"]
    A --> BA["Admin BFF"]

    BW --> GW["Шлюз: общий уровень<br/>маршрутизация · TLS · токен · лимиты"]
    BM --> GW
    BA --> GW

    GW --> K["Каталог"]
    GW --> S["Склад"]
    GW --> Z["Заказы"]
    GW --> O["Оплата"]
    GW --> D["Доставка"]
    GW --> N["Уведомления"]
    GW --> AU["Авторизация"]

    classDef client fill:#FFF2CC,stroke:#D6B656,stroke-width:2px
    classDef bff fill:#DAE8FC,stroke:#6C8EBF,stroke-width:2px
    classDef core fill:#E1D5E7,stroke:#9673A6,stroke-width:3px
    classDef svc fill:#FFFFFF,stroke:#666666
    classDef ours fill:#C51B8A,stroke:#C51B8A,color:#FFFFFF,stroke-width:2px

    class W,M,A client
    class BW,BM,BA bff
    class GW core
    class K,S,Z,O,D,AU svc
    class N ours
```

Розовым — наш сквозной сервис, цвет тот же, что на [карте сервисов](service-map.md) и [Context Map](context-map.md). Рекомендации на схему не вынес: на экране заказа их нет, а плодить стрелки ради полноты не хочется.

Три BFF стоят над общим уровнем, а не вместо него. Всё, что одинаково для любого запроса — проверка токена, лимиты, единый адрес, — живёт этажом ниже и не дублируется трижды. Наверху остаётся только то, что зависит от клиента: какие поля собрать и в какой форме отдать.

## Зум: два входа в сервис уведомлений

```mermaid
graph LR
    Z["Заказы"] --> BR
    O["Оплата"] --> BR
    D["Доставка"] --> BR
    BR["Брокер<br/>события"] --> N

    BW["Web BFF<br/>история в кабинете"] --> GW
    BA["Admin BFF<br/>разбор заявок"] --> GW
    GW["Шлюз"] --> N["Уведомления"]

    N --> P["Внешние провайдеры<br/>SMTP · SMS"]

    classDef bff fill:#DAE8FC,stroke:#6C8EBF,stroke-width:2px
    classDef core fill:#E1D5E7,stroke:#9673A6,stroke-width:3px
    classDef svc fill:#FFFFFF,stroke:#666666
    classDef ours fill:#C51B8A,stroke:#C51B8A,color:#FFFFFF,stroke-width:2px
    classDef ext fill:#FFCCE5,stroke:#B5739D,stroke-width:2px

    class BW,BA bff
    class GW core
    class Z,O,D,BR svc
    class N ours
    class P ext
```

Левый путь — работа: издатели публикуют события, сервис их разбирает и доставляет. Шлюза здесь нет и быть не должно, так решено в [ADR-0002](../adr/0002-notifications-as-separate-service.md).

Правый путь — чтение: история заявок в личном кабинете и разбор в админке. Он идёт через шлюз, как у всех остальных сервисов.

Если нарисовать только первую схему, получится, что уведомления — обычный сервис за шлюзом. Это неправда ровно наполовину, и вторая картинка нужна именно затем, чтобы эту половину не потерять.

## Редактируемые исходники

[`diagrams/entry-layer.drawio`](diagrams/entry-layer.drawio) — та же общая схема в draw.io, если удобнее править мышкой. Mermaid считаю основным: он лежит в тексте, виден прямо на GitHub и правится в том же коммите, что и рассуждение рядом.
