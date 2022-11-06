# Изучение закономерностей, определяющих успешность игр


**Описание проекта:** 

Мы работаем в интернет-магазине «Стримчик», который продаёт по всему миру компьютерные игры. Перед нами исторические данные до 2016 года из открытых источников о продажах игр, оценки пользователей и экспертов, жанры и платформы (например, Xbox или PlayStation). 

**Задачи:**

1. Изучить исходные данные и провести их предобработку.
2. Посчитать суммарные продажи во всех регионах.
3. Выбрать платформы с наибольшими суммарными продажами и несколько потенциально прибыльных платформ.
4. Оценить влияние отзывов пользователей и критиков на продажи.
5. Составить портрет пользователя каждого региона (NA, EU, JP).
6. Проверить гипотезы, что средние пользовательские рейтинги платформ Xbox One и PC одинаковые, а также, что средние пользовательские рейтинги жанров Action и Sports разные.

**Используемые библиотеки и методы:** 

*Pandas*, *NumPy*, *Matplotlib*, *SciPy*, предобработка данных, исследовательский анализ данных, описательная статистика, проверка статистических гипотез.

**Выводы:**

Нами был проведен исследовательский анализ данных видеоигр и обнаружены следующие инсайты:
- количество выпускаемых игр в конце XX века несоизмеримо мало по сравнению с началом XXI
- новые платформы выпускаются примерно спустя 6-8 лет после выпуска прошлого поколения, при этом поддержка предыдущего поколения завершается спустя примерно 10 лет после выпуска
- с учетом количества выпускаемых игр и "срока жизни" платформ, выбираем **актуальный период для анализа в 2013-2016 года**
- в актуальный период лидерами оказываются недавно выпущенные 3 платформы - **Play Station 4, Xbox One и 
Nintendo 3DS**. Определим их как **перспективные в 2017 году**
- на 3DS и PS4 есть "звезды", продажи которых в два раза больше, чем у самых удачных игр у XOne
- при этом, 3DS значительно отстает от конкурентов, т.к. большинство игр имеет высокие продажи именно на Xbox One и еще чуть более высокие на PS4 
- у Playstation и Xbox действительно имеется умеренная корреляция между отзывами критиков и продажами и полное отсутствие зависимости от субъективного рейтинга пользователей, а у Nintendo имеется умеренная зависимость уровня продаж как от рейтинга критиков, так и от рейтинга пользователей
- на примере платформы **PS4** можно сказать, что c большим отрывом **самые высокие продажи** приносят **шутеры**, а за ними идут **спортивные игры**, **в аутсайдерах** явно выделяются **стратегии**. 

Также мы изучили портрета пользователя каждого из трех регионов и выявили следующие основные моменты:
- в Северной Америке примерно одинаково популярны консоли от PlayStation и Xbox, в Европе PlayStation в два раза популярнее, чем Xbox, а в Японии почти половину рынка занимает Nintendo 3DS, а вторую консоли PlayStation 
- в Северной Америке и Европе примерно 2/3 рынка приходится на шутеры с экшн-играми, а в Японии почти половину рынка занимают ролевые игры и около трети экшн-игры  
- в Северной Америке и Европе продажи в зависимости от возрастного рейтинга аналогичны - треть занимают игры 17+, а 40% почти поровну делят между собой игры 10+ и без рейтинга. В то время как в Японии больше половины всех продаж занимают игры без рейтинга, что логично, ведь ESRB предназначен для рецензирования игр США и Канады и на территории Японии не является обязательным.  
    
**Таким образом, собирательный образ популярной игры в Европе и Семерной Америке примерно одинаков - шутер или экшн на PlayStation или Xbox с рейтингом 17+, в то время как в Японии он разительно отличается - ролевая игра на Nintendo 3DS без возрастного рейтинга ESRB.**

Наконец, нами были выдвинуто две гипотезы: 
    
- средний пользовательский рейтинг игр на Xbox One равен среднему пользовательскому рейтингу игр на PC  
- средний пользовательский рейтинг игр жанра Action равен среднему пользовательскому рейтингу игр жанра Sports  

После проведенных t-тестов мы выяснили, что **первую гипотезу нет оснований отвергать**, а **вторая гипотеза скорее всего всё-таки неверна**.