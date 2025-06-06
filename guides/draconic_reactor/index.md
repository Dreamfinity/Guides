# "Гайд по созданию и отладке генератора из Draconic Evolution"

Как собрать правильно генератор и как за ним ухаживать?

### Строим вот такую конструкцию!

<figure markdown="span">  
![](./images/reactor.jpg)
<figcaption><a id="reactor">Рисунок 1</a></figcaption>
</figure>

- Высота стабилизаторов 3 блока (т.е. стабилизатор висит на 4-ом)
- Изыматель энергии реактора ставим на "Contaiment Field (Inverted)"
- В стабилизаторе вставляем топливо "Пробуждённый драконовый блок" в количестве 8 шт. (вроде как)
- Изходя из [рисунка 1](#reactor), нижний передатчик выставляем как на [рисунке 2](#lower_controller), а верхний как на [рисунке 3](#upper_controller)


    <figure markdown="span">
    ![](./images/lower.png)
    <figcaption><a id="lower_controller">Рисунок 2</a></figcaption>
    </figure>


    <figure markdown="span">
    ![](./images/upper.png)
    <figcaption><a id="upper_controller">Рисунок 3</a></figcaption>
    </figure>

### Остается дело за малым.
- В стабилизаторе есть кнопка "Charge reactor", нажимаем! Ждите покуда наберутся, "Температура = 2000, Сила поля = 50%, Заряд = 50%" 
- После зарядки реактора, появится кнопка "Activate". Нажимаем и ждём, пока вся статистика не будет похожа как на [рисунке 4](#statistics)

<figure markdown="span">
![](./images/stats.png)
<figcaption><a id="statistics">Рисунок 4</a></figcaption>
</figure>

Вроде ничего не забыл... Выроботка RF/t моментальная и быстрая, за секунду около 10 000 000 RF, помимо реактора есть и другие источники энергии, так что жить только за счёт реактора, НЕ СОВЕТУЮ!

P.S. Для зарядки реактора используется не только топливо, но и RF энергия
