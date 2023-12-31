# Module №2 | Разметка данных
## Задача
В модуле-1 была проведена сборка датасета дорожных знаков. Для этого использовался датасет [Russian traffic sign images dataset](https://www.kaggle.com/datasets/watchman/rtsd-dataset). Также был проведен веб-скрейпинг видео с YouTube с последующим формированием снимков кадров каждого видео.

В этом модуле необходимо провести аннотацию дорожных знаков на изображениях, которые были получены из видеозаписей YouTube, записанных на видеорегистраторы во время движения.

## Данные
**Исходные данные:** 
- изображения с видеорегистраторов, 100 шт:
    - размеченные изображения с датасета RTSD, 50 шт;
    - неразмеченные изображения, полученные из видеозаписей с YouTube, 50 шт.
- файл `labels.txt` со списком возможных меток классов, 155 классов;
- файл `ПДД._Дорожные_знаки.pdf` с каталогом дорожных знаков.

**Инструмент для разметки:** CVAT (Computer Vision Annotation Tool) \
**Выходные данные:** json-файл с bounding box дорожных знаков на изображениях


## Разметка и оценка качества
В разметке дорожных знаков участвовал один разметчик. Для оценки доверия разметчику и качества составленного технического задания разметчику были переданы два набора изображений:
- набор с разметкой, составленный авторами датасета, 50 шт;
- набор, требующий разметки, 50 шт.

С помощью первого набора будут оценены метрики качества разметки.

### Подход-1
**Сформулируем задачу, которую решал разметчик:** \
Задача похожа на задачу классификации с несколькими метками (multi-label classification). Одному изображению присваивается несколько меток в зависимости от количества обнаруженных на нем дорожных знаков. Поэтому, имея разметку, составленную авторами датасета, можно оценить работу разметчика (по аналогии, работу классификатора), используя метрики качества задачи multi-label classification.

#### Exact Match Ratio
$$\text{Exact Match Ratio, MR} = \frac{1}{n} \sum_{i=1}^{n} I(y_{i} = \hat{y_{i}})$$

#### 0/1 Loss
$$\text{0/1 Loss} = \frac{1}{n} \sum_{i=1}^{n} I\left(y_{i} \neq \hat{y_{i}} \right)$$

#### Accuracy
$$\text{Accuracy} = \frac{1}{n} \sum_{i=1}^{n} \frac{\lvert y_{i} \cap \hat{y_{i}}\rvert}{\lvert y_{i} \cup \hat{y_{i}}\rvert}$$

#### Hamming Loss
$$\text{Hamming Loss} = \frac{1}{n L} \sum_{i=1}^{n}\sum_{j=1}^{L} I\left( y_{i}^{j} \neq \hat{y}_{i}^{j} \right)$$

#### Recall
$$\text{Recall} = \frac{1}{n} \sum_{i=1}^{n} \frac{\lvert y_{i} \cap \hat{y_{i}}\rvert}{\lvert y_{i}\rvert}$$

#### Precision
$$\text{Precision} = \frac{1}{n} \sum_{i=1}^{n} \frac{\lvert y_{i} \cap \hat{y_{i}}\rvert}{\lvert \hat{y_{i}}\rvert}$$

#### F1-Measure
$$F_{1} = \frac{1}{n} \sum_{i=1}^{n} \frac{2 \lvert y_{i} \cap \hat{y_{i}}\rvert}{\lvert y_{i}\rvert + \lvert \hat{y_{i}}\rvert}$$

Недостаток такого подхода заключается в том, что метрики не учитывают то, сколько раз метка была отнесена к данному изображению, она учитывает только факт того, что метка была присвоена.

### Подход-2
Можно заметить, что в наборе, размеченном авторами датасета, всего 118 строк о дорожных знаках, в то время как после разметки данного набора разметчиком в результирующей таблице 156 строк. Это говорит о том, что разметчик разметил дорожный знак, который не был размечен в исходном датасете, либо разметчик разметил мусор (см. таблицы ниже). 

Способы решения проблемы:
- обратиться к нескольким разметчикам, сравнить согласованность результатов;
- проверить на наличие мусора вручную. 

<img src='imgs/test_frame.png' style='width:500'></img>
<img src='imgs/annot_frame.png' style='width:500'></img>

Проведем только численный анализ меток в тестовом наборе и в наборе, полученном от разметчика.

## Обратная связь и выводы
### Обратная связь
По результату разметки данных разметчик хорошо оценил оформление и подробное описание технического задания. 

### Выводы
Таблица `Оценка качества разметки разметчика как задача multi-label classification`

|            | Exact Match Ratio | Hamming loss | Recall | Precision | F1 Measure |
|------------|-------------------|--------------|--------|-----------|------------|
| Score      |       0.46        |    0.006     |  0.75  |    0.9    |    0.8     |
|            |                   |              |        |           |            |

С точки зрения оценки качества разметки подходом-1 получены неплохие результаты, хоть и существует небольшая рассогласованность в разметке. 

Таблица `Согласованность меток тестового набора и меток, полученных разметчиком`

|                | Всего меток* | Кол-во совпадений** | Не найдено*** |
|----------------|--------------|---------------------|---------------|
| Тестовый набор |     118      |          107        |       11      |
| Разметчик      |     156      |          107        |       49      |
|                |              |                     |               |
    * кол-во опознаных дорожных знаков
    ** кол-во дорожных знаков, согласованных в тестовом наборе и наборе разметчика
    *** (Всего меток - Кол-во совпадений), кол-во меток, не найденных в противоположном наборе

Подход-2 позволяет численно оценить количество несовпадений размеченного набора и результат разметки разметчика. По таблице видно, что в большинстве случаев разметчик правильно аннотирует данные. В 11 знаках присутствует рассогласованность, 49 - 11 = 38 являются мусором, либо не были размечены в тренировочном датасете. 

Таким образом, в результате работы проведена аннотация дорожных знаков на изображениях, которые были получены из видеозаписей YouTube, записанных на видеорегистраторы во время движения и оценено качество разметки данных разметчиком.