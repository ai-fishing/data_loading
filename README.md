# Project brief
## Общая задача:
Существует набор данных измерений по определенному района моря. Как, использую техники Datascience, предсказывать, в какой точке района может быть рыба в нужном количестве; определить примерный объем; оценить динамику. 
## Зачем решать:
Рыбаки теряют большие деньги, когда неправильно прогнозируют промысловую ситуацию. Например, лишний день «пустого» пробега судна – 1-4 миллиона руб.
## Обоснование, почему предсказание возможно:
Рыба всегда там, где определенное сочетание факторов (признаков), указывающих на нужное количество пищи и комфортные условия, это установлено научно. Правда, они для разных районов, видов рыбы и сезона различаются. 
При этом опытные рыболовы обычно более-менее точно выходят на места обитания косяков, на основе своего опыта.
Это значит, скрытая зависимость есть, надо ее найти моделями машинного обучения.
Задача максимум: находить именно точки (точнее, небольшие квадраты) местоположений больших скоплений рыбы.
## Признаки
Основные признаки, по которым можно оценить комфортность локации и наличие достаточного количества пиши (а также динамику смещения куда-то) и которые реально измерют:
1.	ТПО (поверхности океана)
2.	Цвет воды
3.	Прозрачность
4.	Уровень кислорода
5.	Соленость
6.	Уровень моря
7.	Скорость и направление приводного ветра
8.	Батиметрия (рельеф дна, глубина).
9.	Температура воздуха на высоте 2 м
10.	Состояние поверхности (длина, высота и направление волн)
11.	Концентрация хлорофилла

Эти параметры меняются слабо в пространстве и времени. 
В пространстве достаточно задавать шаг изменения 1 км, по времени – раз в сутки.

Есть ли еще какие-то полезные фичи? Пока непонятно, надо пробовать. Например, мне кажется, расстояние до берега должно иметь значение.

## Разметка данных

Достаточно сложный вопрос. Надо каждый квадрат на данный период (например, сутки) пометить как «Да/Нет». Для этого надо для обучающей выборки знать места, где видели и ловили рыбу.

Такие данные рыболовы стараются не сообщать. Возможно, такие данные сдают в госорганы для отчетности, но это не точно (стоит проверить), и непонятно, можно ли их оттуда добыть.

Второй вариант – по косвенным признакам. Рыболовные суда имеют разные типы движения при спокойном ходу и при ловле (меняется направление, скорость, определенный рисунок траектории). По морским соглашениям каждое судно передает параметры движения через систему AIS. То есть, имея траектории, можно по ним обучиться и проставить места лова по историческим данным.  
Проблема: в свободном доступе такие данные не распространяются (за деньги будет слишком дорого). Хотя варианты есть.

## Алгоритмы

Очень непростая часть. Явно должен быть набор алгоритмов, одним не решается.
Если про поиск скоплений по размеченным данным, то там есть :

- связь между соседними квадратами (точками);
- зависимость по времени между соседними сутками.

Ничего готового похожего я пока не находил. Из близких вариантов:
- считать морской район чем-то вроде многоканального изображения, травить его нейросетями для изображений;
- попробовать Conditional Random  Fields;

Чтобы было с чего начать, можно начать с упрощения: брать небольшие кусочки моря за сутки и попробовать GBM.

Для обучения по траекториям судов по данным AIS уже кое-что делали до нас. Похожая задачка была на Topcoder. Плюс можно посмотреть по статейкам (у меня есть):

1)	de Souza EN, Boerder K, Matwin S, Worm B (2016) Improving Fishing Pattern Detection from Satellite AIS Using Data Mining and Machine Learning. PLoS ONE 11(7): e0158248. doi:10.1371/journal.pone.0158248
2)	Natale F, Gibin M, Alessandrini A, Vespe M, Paulrud A (2015) Mapping Fishing Effort through AIS Data. PLoS ONE 10(6): e0130746. doi:10.1371/journal.pone.0130746
3)	Hu, Baifan, Xiang Jiang, Erico N. de Souza, Ronald Pelot and Stan Matwin. “Identifying fishing activities from AIS data with Conditional Random Fields.” 2016 Federated Conference on Computer Science and Information Systems (FedCSIS) (2016). DOI: 10.15439/2016F546
4)	Xiang Jiang, Daniel L Silver, Baifan Hu, Erico N de Souza, and Stan Matwin. Fishing activity detection from ais data using autoencoders. In Canadian Conference on Artificial Intelligence, pages 33–39. Springer, 2016.
5)	F. Mazzarella, M. Vespe, D. Damalas and G. Osio, "Discovering vessel activities at sea using AIS data: Mapping of fishing footprints," 17th International Conference on Information Fusion (FUSION), Salamanca, 2014, pp. 1-7.
Может, на нейросетках можно что-то даже и лучше сделать.

## Метрики

Надо определить, по чему оптимизируем. Можно считать все несовпадения, можно учитывать отклонение от «правильного» квадрата и т.п. 

## Сборка датасета, работа с данными
Надо будет определять границы промысловых зон, они международно согласованы.
Это где то здесь есть:
http://www.fao.org/fishery/area/search/en
По основным параметрам есть открытые источники, можно работать.
Основные источники (по океану):
https://earthdata.nasa.gov/discipline/ocean
http://marine.copernicus.eu
http://www.remss.com
https://ladsweb.nascom.nasa.gov/search
http://www.smos-sos.org
Ветер (пример)
ftp://ftp.ifremer.fr/ifremer/cersat/products/gridded/MWF/L3/ASCAT/Daily/Netcdf/2016
Обычно можно скачать по ftp/http файлы по заказу на день/период, формат будет NetCDF,HDF скорее всего.
Остальные параметры надо поискать еще будет.
Надо еще посмотреть на ограничения на скачивание, сколько можно за раз взять.

Для каждого вида данных может быть своя сетка по пространству, надо перепривязывать к единой 1 км на 1 км.

## Данные по траекториям судов AIS (Automatic Identification System)

По AIS есть поставщики данных, например, marinetraffic, vesselfinder и пр. Большое количество данных, особенно исторических, можно получить только за деньги. В свободном доступе дают понемножку.
Какие варианты:
- что то есть в свободных датасетах для обучения, но надо выкапывать;
- где можно точно взять датасет:  Сайт некоммерческой организации globalfishingwatch.com
Но там данные анонимизированы, надо аккуратно смотреть, как их привязать к местности.

Также учтем, что разные типы судов (траулер, сейнер..) надо распознавать по-разному, поэтому в данных AIS придется смотреть идентификатор судна, а по нему определять тип, например, по данным морского регистра. 

Или делать свою базу (в одном районе моря обычно одни и те же суда пасутся). 

## В плане подготовки данных можно идти таким путем:
- Добиваем информацию по источникам основных данных, где что в каком формате; 
- Смотрим, где у нас может быть хорошо с данными AIS
- Определяем тест-район, тип рыбы;
- Определяем структуру датасета на район
- Мастерим датасет
Т.е. нам потребуется модуль для скачивания по параметрам; обработчик данных; модуль для работы с датасетом. Данные AIS и MODIS я немного разбирал, будет попроще.
## Среда для разработки и инфраструктура
- Для разработки и тестирования алгоритмов – стандартный набор Python для датасайенса: pandas, numpy, scikit, pytorch/tensorflow/keras…
- Для геопроцессинга - ? (gdal, shapely…)
- СУБД - ? (например, postgress)
- Данные могут быть большие, надо где-то в облаке держать для совместного пользования;
- Тогда для запуска тестов нужен также и виртуальный сервак. (Надо посмотреть, влезет ли все нужное на бесплатный лимит).
- Мажорные версии предлагаю держать на гите на ai-fishing
- Для визуализации данных с сервера предлагаю состряпать веб-сервис (например, Django+react, но не обязательно) с кнопками управления для отображения и фильтров (потом можно из этого делать прототип пользовательской морды).








