# RU

## Задание 1
``` python
'''Класс CPStrategy наследуется от StaticStrategy '''
class CPStrategy(StaticStrategy):
    _seed_type = 'product_id' # private переменная, которая отвечает, по видимому за то, какая колонка будет индентификатором товара
    _counter_mapping = {
        'product_views': 'coview_counts',
    } # private переменная, словарь, ключ - то что хотим посчитать (можно назвать метрикой? или как далее название таблицы, за которой закреплен определенный метод подсчета), значение - счетчик или название счетчика (метода)

    # функция инициализации класса
    def __init__(self,
                 dataset: 'lib.datasets.functional.DataSet', # аргумент, подразумевающий тип данных DataSet из какой-то либы
                 strategy_hash: str, # все три этих стороковых аттрибута, по видимому, отвечают за определенную стратегию, т.е. идентифицируют ее однозначным образом
                 strategy_name: str,
                 revision: str,
                 # params
                 table_name: str, # Название таблицы, откуда будут браться данные.
                 use_counter: bool = True, # bool использовать или нет счетчик
                 sess_min_len: int = None, # минимальная длина сессии? окна
                 sess_max_len: int = None, #
                 count_threshold: int = 5, # пороговое значение счетчика
                 topk: int = 25): # какое количество топовых значений необходимо взять, т.е. порог для отбара top N значений
        self.debug('Building CP strategy!') # обращение к методу для логирования уровня debug. вывод сообщения "Building CP strategy"
        valid_tids = dataset.get_valid_products() # выбор из датасета товаров, который удовлетворяют каким-либо условиям

        # если флаг use_counter = True и имя таблицы table_name есть в словаре, то выполняется
        if use_counter and table_name in self._counter_mapping:
            """
             def get_coviews(self, age: int = 120, stop: bool = True):
                df = self.read_csv('data/co_product_views', ['tracking_id', 'next_tracking_id', 'count'])
                if df is None:
                    return None
                df['count'] = df['count'].astype(int)
                return df
            """
            counter = getattr(dataset, self._counter_mapping[table_name]) # с помощью getattr из dataset достается нужный метод счетчика, в нашем случае если будет table_name = product_views, то вернется название метода coview_counts

            # датафрейм создается с помощью генератора списков, который достает seed, rec, score и сравнивая, чтобы score был больше трешхолда, а так же seed и rec были в допустимых пределах, после чего помещаются они в соответствующие колонки
            strategy_df = pd.DataFrame([(seed, rec, score) for seed, cntr in counter.items()
                                        for rec, score in cntr.most_common(topk)
                                        if score > count_threshold
                                        and seed in valid_tids
                                        and rec in valid_tids],
                                       columns=['seed', 'recs', 'score'])
```
Отдельно рассмотрим ```def get_coviews()```
```python
    # метод, на вход два именованных аргумента, со значениями по умолчанию, вовзращает dataframe, к слову лучше бы это указать
    def get_coviews(self, age: int = 120, stop: bool = True) -> [None, pd.DataFrame]:
        df = self.read_csv('data/co_product_views', ['tracking_id', 'next_tracking_id','count']) # из csv файлика читается датафрейм в колонки с обозначенными именами
        # если датафрйм пустой метод возвращает None
        if df is None:
            return None
        df['count'] = df['count'].astype(int) # приведение типа данных в колонке count к целому числу
        return df # возвращается датафрейм
```
Более подробно про принцип рекомендаций:
```python 
[(seed, rec, score) for seed, cntr in counter.items()
                    for rec, score in cntr.most_common(topk)
                    if score > count_threshold and seed in valid_tids and rec in valid_tids]
```
Тут происходит итерация по каждому товару (seed) и его счетчику (cntr), который хранит товары, совместно просмотренные с seed.
Далее, для каждого товара, выбирается топ-K самых популярных совместных товаров с помощью cntr.most_common(topk), где rec — это рекомендованный товар, а score — частота совместного просмотра.

## Задание 2

Решение в файлике task2.ipynb

## Задание 3


## Задание 4

## Задание 5

## Задание 6
