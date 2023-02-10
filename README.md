# Feedback-Prize-English-Language-Learning
Описание и код решения соревнования [Feedback Prize - English Language Learning](https://www.kaggle.com/competitions/feedback-prize-english-language-learning)
**Результат**: 54/2654 (top 3%)

## Постановка задачи
По набору эссе учеников 8-12 классов построить модель, оценивающую эссе по 6 критериям: грамматика, пунктуация, словарный запас, фразеология, логическая связность текста, языковая конвенция. Оценки эссе в обучающей выборке варьируются от 1 до 5 с шагом 0.5.
Решение оценивалось по метрике MCRMSE (усредненное MSE по 6 критериям)

## Стратегия валидации
При проведении экспериментов модели оценивались на 4 x MultilabelStratifiedKFold. При отправке предсказаний модель обучалась на 10 x MultilabelStratifiedKFold

## Решение
### Лучшая модель
*  deberta-v3-base
*  Attention based pooling
*  Dropout: 0
*  Epochs: 4
*  batch_size: 8
*  Grouped Layer-wise Learning Rate Decay
   *  for 0-3 layers: lr/1.6
   *  for 4-7 layers: lr
   *  for 8-11 layers: lr*1.6
*  Learning rate: 2e-5
*  Weight decay: 0.01
*  Optimizer: AdamW
*  Sheduler: cosine

### Модели, используемые в финальном ансамбле
| model          |     pooling        |      CV      | public LB    |  private LB  |
| -------------  |:------------------:|:------------:|:------------:|:------------:|
| deberta-v3-base| Attention          |    0.4511    |    0.4511    |    0.4392    |
| deberta-v3-base| Concatenation 4 last layers              |    0.4547    |    0.4432    |    0.4411    |
| roberta-large| Mean               |    0.4618    |   0.4439    |    0.4424    |
| deberta-base| Mean               |    0.4637    |    0.4479    |    0.4443    |
| deberta-v3-base| CNN              |   0.4577    |    0.4444    |    0.4446    |
| funnel-transformer-large| Mean               |   0.4687    |    0.4524    |    0.4455    |
| albert-base-v2| Mean               |   0.4713    |    0.4549    |    0.4540    |
| xlmroberta-base| Mean               |   0.4765    |    0.4578    |    0.4545    |
| xlnet-base-cased| Mean               |    0.4908    |    0.4671    |    0.4591    |
| deberta-v3-large| Mean               |     0.4548   |    -    |    -    |
| deberta-v3-large| Mean               |    0.4569    |    -    |   -    |
| deberta-v2-xlarge| Mean               |    0.4604   |    -    |    -    |
| deberta-v2-xlarge-mnli| Mean               |    0.4675    |    -    |    -    |

### Финальный ансамбль
| method         |      CV      | public LB    |  private LB  |
| -------------  |:------------------:|:------------:|:------------:|
| Nelder-Mead    |    0.4440   |    0.437255   |    0.0.435666    |

## Не сработало
*  Добавление специального токена, обозначающего новый абзац (/n/n)
*  Переинициализация последнего слоя / нескольких последних слоёв
*  Обучение SVR на эмбеддингах предобученных моделей (решение, основанное на ансамбле трансформеров и SVR заняло бы ~ 90 место в частной таблице лидеров)
*  Использование признаков библиотеки readability при построении моделей второго уровня
*  Использование признаков, извлеченных из текстов (количество слов, уникальных слов, предложений, знаков препинания...) при построении моделей второго уровня
*  Стэкинг. Линейная регрессия уступала оптимизации весов методом  Нелдера — Мида на 0.02 (по метрике MCRMSE), LightGBM - на 0.03
*  Постобработка предсказаний (округление ухудшало метрику на 0.001)

## К сожалению, не хватило времени на 
*  настройку AWP(Adversarial Weight Perturbation)
*  псевдо-лейблинг
