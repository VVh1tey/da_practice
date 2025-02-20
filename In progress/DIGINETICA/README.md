# RU

## Структура:
```plaintext
DIGINETICA/
├── README.md -- файл с описанием
├── Answers.md -- файл, содержащий текстовые ответы на те вопросы, к которым это необходимо
└── README.md
```

## Условие задачи

### 1. Разбор кода стратегии рекомендаций
Ниже приведен код для расчета одной из стратегий рекомендаций. Необходимо:
Объяснить, что делает данный код.
Пояснить, как работает каждая его часть.
``` plane text
class CPStrategy(StaticStrategy):
    _seed_type = 'product_id'
    _counter_mapping = {
        'product_views': 'coview_counts',
    }

    def __init__(self,
                 dataset: 'lib.datasets.functional.DataSet',
                 strategy_hash: str,
                 strategy_name: str,
                 revision: str,
                 # params
                 table_name: str,
                 use_counter: bool = True,
                 sess_min_len: int = None,
                 sess_max_len: int = None,
                 count_threshold: int = 5,
                 topk: int = 25):
        self.debug('Building CP strategy!')
        valid_tids = dataset.get_valid_products()
        if use_counter and table_name in self._counter_mapping:
            """
             def get_coviews(self, age: int = 120, stop: bool = True):
                df = self.read_csv('data/co_product_views', ['tracking_id', 'next_tracking_id', 'count'])
                if df is None:
                    return None
                df['count'] = df['count'].astype(int)
                return df
            """
            counter = getattr(dataset, self._counter_mapping[table_name])
            strategy_df = pd.DataFrame([(seed, rec, score) for seed, cntr in counter.items()
                                        for rec, score in cntr.most_common(topk)
                                        if score > count_threshold
                                        and seed in valid_tids
                                        and rec in valid_tids],
                                       columns=['seed', 'recs', 'score'])
```
#### Ожидания:

- Поясните работу ключевых частей кода: наследование, назначение переменных, фильтрация данных и генерация DataFrame.
- Расскажите, как код строит рекомендации для товаров на основе просмотров (co-viewing).
- Опишите, каким образом фильтруются и формируются финальные рекомендации.

### 2. Построение рекомендаций на основе популярности товаров

**Задача**: скачать датасет DIGINETICA и написать код, который строит рекомендации к каждому товару на основании популярности. На выходе должен получиться CSV-файл формата:

```
seed_item_id, recommended_product_id, score

```

#### Ожидания:

- Напишите код, который:
    - Загружает датасет.
    - Рассчитывает популярность товаров (например, на основании количества просмотров или покупок).
    - Создает рекомендации для каждого товара.
    - Сохраняет результат в CSV.

### 3. Присвоение статуса доступности товара

Для определения доступности товара к покупке мы используем YML фиды:

- Есть глобальный фид, содержащий все товары из каталога.
- Есть региональные фиды для каждого региона, которые указывают доступность товара в конкретном регионе.

**Вопрос**: как присвоить статус доступности товару, если самого товара нет в региональном фиде?

#### Ожидания:

- Описание процесса присвоения статуса товару в случае отсутствия региональных данных.

---

### 4. Фильтрация товаров по атрибуту "Ноты аромата"

Есть стратегия фильтрации рекомендуемых товаров по атрибуту "Ноты аромата". Необходимо отфильтровать рекомендуемые товары по следующим параметрам:

- У seed-товара атрибут "Ноты аромата" = роза, пачули, кедр.
- Жесткость фильтрации: **DEFAULT** — полное текстовое вхождение всех значений атрибута seed-товара среди значений атрибута рекомендуемых товаров.
- `include_no_attribute = false` — не включать товары, у которых отсутствует атрибут для фильтрации.
- Разделитель значений внутри одного атрибута — запятая.

#### Товары для фильтрации:

- **Товар А**: роза
- **Товар Б**: пачули, кедр
- **Товар С**: кедр, пачули, роза, ладан
- **Товар Д**: ладан, роза
- **Товар Г**: атрибут отсутствует

#### Ожидания:

- Выберите из перечисленного списка товары, которые пройдут фильтрацию.
- В ответе укажите заглавные буквы товаров, которые успешно прошли фильтрацию.

### 5. Фильтрация товаров по атрибутам "Сахар" и "Цвет"

Есть стратегия фильтрации рекомендуемых товаров по атрибутам "Сахар" и "Цвет":

- У seed-товара 1: Сахар = полусладкое; Цвет = красное.
- Жесткость фильтрации: **HARD** — полное текстовое совпадение всех значений атрибута seed-товара и атрибута рекомендуемых товаров.
- `include_no_attribute = false` — не оставлять товары без атрибута для фильтрации.
- `is_all_attrs_include = true` — совпадение должно быть по всем атрибутам.

#### Товары для фильтрации:

- **Товар А**: Сахар = полусладкое; Цвет = белое
- **Товар Б**: Сахар = сухое; Цвет = красное
- **Товар С**: Сахар = полусладкое; Цвет = красное
- **Товар Д**: Сахар = полусладкое; атрибута Цвет нет
- **Товар Г**: Сахар = полусладкое, сладкое; Цвет = красное

#### Ожидания:

- Выберите те товары, которые пройдут фильтрацию для **seed-товара 1**.

Также необходимо провести фильтрацию для **seed-товара 2**, у которого только атрибут "Сахар" = сухой, а атрибута "Цвет" нет:

#### Товары для фильтрации:

- **Товар Е**: Сахар = сухой; Цвет = белый
- **Товар Ж**: Сахар = полусухой; атрибута Цвет нет
- **Товар З**: Сахар = сухой; Цвет = розовый
- **Товар И**: Сахар = сухой; атрибута Цвет нет
- **Товар К**: Сахар = сухое; атрибута Цвет нет

#### Ожидания:

- Укажите, какие товары прошли фильтрацию для **seed-товара 2**.

### 6. Анализ ответа API

Дан запрос к API:

```
<https://recs.diginetica.net/recs?apiKey=3847X11479&placements=item_page.alternatives%7Citem_page.cross_sell&showOnlyAvailable=true&productId=19760302811&fullData=true>

```

#### Ожидания:

- Проанализируйте ответ API.
- Опишите, что можно узнать о первом товаре из предоставленного ответа.

---

### Ожидаемый формат ответа:
Ответы должны быть структурированы, содержать четкие пояснения и выводы для каждой задачи.