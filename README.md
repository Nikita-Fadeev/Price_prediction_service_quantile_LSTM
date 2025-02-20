# Microservice for offering prices for 1000 SKU products based on user requests.
## The task of maximizing sales with maximum margin.

- A microservice client has been developed that offers a `price` for the combination of `dates`, `user_id`, and `SKU`. The service `price_service.ipynb` operates according to the following algorithm: 
    1. A GET request receives data from the server with `date`, `user_id`, and `SKU`. 
    1. The price of the requested `SKU` product is selected for each `user_id`. 
    1. A POST response publishes the proposed `prices`. 
    1. The purchase fact `bougth` is received by the service via a GET request, and the `update_reward` method is used to adjust the `price` selection function for each `user_id`.
- The service uses the `MultiArmBandit` algorithm to select prices for each `user_id` in accordance with maximum profit. Price selection is made from a discrete range of prices for each `SKU` product, calculated for each week. The `price` varies according to the 5th, 50th, and 95th percentiles of the `markup`.
- The following pipeline was used to predict the `price` range `price_predictions.ipynb`: 
    1. Quantile LSTM models were trained for each `SKU` product on purchase quantity data. These models are capable of predicting the demand range `num_purchase`. 
    1. Quantile regression models were also built for each `SKU` product `QuantileRegressionModel`, describing the relationship between demand `num_purchase` (purchase quantity) and `markup` for each `SKU` product. 
    1. Knowing the purchase `cost_price` for the target dates (December 2019), discrete `price` ranges were predicted for each `SKU` product. 
    1. For each week of December, for each `SKU` product, the 5th, 50th, and 95th percentiles of purchase quantity were predicted. Based on the purchase quantity, the 5th, 50th, and 95th percentiles of `markup` were found, and from the discrete range of `markups` and purchase `costs_price` for December, the price range for each product was determined `[price_pred_q5, price_pred_q50, price_pred_q95]`.

Notes:

- As a strategy for multi-armed bandits, the epsilon-greedy learning algorithm was chosen `EpsGreedy(Strategy)`.
- During EDA, each of the 1000 `SKUs` was clustered using the DBSCAN algorithm. Clustering was based on embeddings of price time series. The clustering result was not used for training but was utilized for visualization.
- For building quantile LSTM models, a search for the length of time windows was conducted `SKU_lag[SKU]`. The search and selection were performed using the autocorrelation function. A suitable lag was considered a window where the module of the autocorrelation function was less than 0.1 divided by the number of counts.
- Data on holidays `df_holidays` were used exclusively for visualization.
- For `SKUs` for which it was not possible to build LSTM quantile regressions `SKUs_remain`, markups were obtained by averaging markups over a known period.
- During the training of quantile LSTM models, the quality metric was the Intersection Over Union (IoU) coefficient—the ratio of the intersection area of the predicted and true purchase quantity range to the area of the figure describing these ranges. The true range equals the true value +- the square root of the standard deviation of the purchase quantity for a specific `SKU` over the training period.

# Микросеврис для предложения цен 1000 SKU товаров по запросам пользователей. 
## Задача максимизации продаж с максимальной маржой.

- Разработан микросервисный клиент, который для связки `[dates, user_id, SKU]` предлагает цену `price`. Сервис `price_service.ipynb` работает по следующему алгоритму:
    1. GET запрос получает данные от сервера `[dates, user_id, SKU]`
    1. Выбирается цена запрошенного `SKU` товара для каждого пользователя `price`
    1. POST ответ публикует предложенные цены 
    1. Факт покупок `bought` сервис получает GET запросом и с помощью метода `update_reward` осуществляется настройка функции подбора цен для каждого `user_id` 
- Сервис посредством алгоритма многоруких бандитов `MultiArmBandit` для каждого `user_id` пользователя осуществляет подбор цены в соответствии с максимальной прибылью. Выбор цен осуществляется из дискретного диапазона цен для каждого `SKU` товара, рассчитанного на каждую неделю. Цена варьируется в соответствии 5, 50 и 95 перцентилем наценки `markup`
- Для предсказания диапазона цен был использован следующий пайплайн `price_predictions.ipynb`:
    1. Для каждого `SKU` товара были обучены квантильные LSTM модели `SKU_LSTM_quantille` на данных кол-ва покупок `num_purchase`. Т.е. были получены модели способные предсказать диапазон спроса. 
    1. Для каждой единицы товара `SKU` так же были построены квантильные регрессии `QuantileRegressionModel` описывающие связь спроса (кол-во покупок) `num_purchase` и наценки `markup` на единицу товара `SKU`.  
    1. Зная стоимость закупки на целевые даты (декабрь 2019 года) были предсказаны дискретные диапазон цен на каждый `SKU` товар.
    1. Т.е. на каждую неделю декабря, для каждого товара `SKU`, было предсказан 5, 50, 95 перцентиль кол-ва покупок `num_purchase`, на основании кол-ва покупок найдены 5, 50, 95 перцентиль наценки на товар `markup`, из дискретного диапазона наценок и стоимости закупок `costs_price` на декабрь был найден диапазон цены на каждый товар `[price_pred_q5, price_pred_q50, price_pred_q95]`.
    
Примечания:
- В качестве стратегии для многоруких бандитов был выбран эпсилон-жадный алгоритм обучения `EpsGreedy(Strategy)`.
- При проведении EDA каждый из 1000 `SKU` был кластеризован с помощью алгоритма DBSCAN. Кластеризация происходила на основании эмбеддингов временных рядов цен. Результат кластеризации не был применен для обучения, однако использовался при визуализации.
- Для построения квантильных LSTM моделей был осуществлен поиск длинны временных окон `SKU_lag[SKU]`. Поиск и подбор осуществлялся с помощью автокорреляционной функции. Подходящим лагом считалось окно, при котором модуль автокорреляционной функции был менее 0.1 / кол-во отсчетов.
- Данные по праздничным дням `df_holidays` использовались исключительно при визуализации.
- Для `SKU` для которых не удалось построить LSTM квантильные регрессии `SKUs_remain` наценки были получены усреднением наценок за известный период
- При обучении моделей квантильной LSTM модели метрикой качества обучения служил коэффициент Intersection Over Union (IoU) - отношение площади пересечения предсказанного и истинного диапазона кол-ва покупок `num_purchase` к  площади фигуры описывающей эти диапазоны. Причем истинный диапазон = истинное значение +- корень из стандартного отклонения кол-ва покупок конкретного `SKU` за тренировочный период.
