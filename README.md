# ОЦЕНКА РЕЛЕВАНТНОСТИ ТЕСТИРОВАНИЯ И РАНЖИРОВАНИЯ МОДЕЛЕЙ В ЗАВИСИМОСТИ ОТ ОБЪЕМА ДАТАСЕТА

Методы машинного обучения все чаще используются в различных областях жизнедеятельности. Ежегодно множество научных коллективов разрабатывают новые распознающие модели, соревнуясь при этом в показателях качества на открытых датасетах. В некоторых задачах показатели точности давно превысили 99\%, при этом лучшие в таблице ранжирования модели зачастую отличаются между собой на сотые доли процентов. Принимая в расчет объемы датасетов, резонным становится вопрос о релевантности оценки качества и достоверности ранжирования различных распознающих моделей.

## Математическая модель

Итак, мы говорим, что бенчмарк задает задачу, под этим мы понимаем, что существует вероятностное пространство $( \Omega ,\mathcal{F} ,P)$, множество объектов, предполагаемое бенчмарком, с сигма-алгеброй $(O,\mathcal{F}_{O})$ и счетное число  измеримых и независимых в совокупности отображений $\xi _{i} :\Omega \rightarrow O$ с совпадающими распределениями. Тестовая часть датасета $O_t$ это конечное множество независимых реализаций случайных элементов $\xi _{i}$. На элементах множества $O$ задано измеримое отображение $v$ в пространство возможных ответов на объекте $K$, которое представляет собой сопоставление объектам правильных ответов. Модель $m$ также является измеримым отображением из  $O$ в $K$, введем функцию $g_{m} :O\rightarrow \{0,1\}$, 
\begin{equation}
    g_{m}( x) =I[ m( x) =v( x)],
\end{equation}
где $I[\cdot]$--функция индикатор, принимающая 1, если выражение верно и 0 иначе. Тогда Accuracy $A^{t}_{m}$ модели $m$ на $O_t$ вычисляется как
\begin{equation}
    A_{m}^{t} =\sum _{x\in O_{t}} g_{m}( x) /\left| O_{t}\right|,
\end{equation}
где $\left| O_{t}\right|$ объем тестовой выборки. Легко заметить, что $g_m$ измерима. Исходя из вышесказанного, естественно дать определение Accuracy $A_m$ модели $m$ в задаче как
\begin{equation}
    A_{m} =E_{P}[ g_{m}( \xi _{0})].
\end{equation}
Далее мы будем предполагать, что случайные величины $\{g_{m}( \xi _{i})\}$ для всех рассматриваемых моделей независимы в совокупности.  

## Принадлежность модели к высококачественным или низкокачественным
Итак, пусть есть условно высококачественные модели, для них верно, что  $A_{m} \geqslant p_{0}$ и низкокачественные с $A_{m} \leqslant p_{1} < p_{0}$, где $p_0$, $p_1$ фиксированы. Модели с $A_m \in (p_1, p_0)$ неклассифицируемые и для нас не будет критичным отнести такую модель к любому из классов. 

Предположим, необходимо уметь классифицировать любую модель с $A_{m} \notin ( p_{1} ,p_{2})$ с ошибкой первого рода на уровне $\alpha < 0.5$ и ошибкой второго рода на уровне $\beta < 0.5$. При проверке гипотезы $H_{0} :A_{m} \geqslant p_{0}$, против альтернативы $H_{1} :A_{m} \leqslant p_{1} < p_{0}$, наибольшую сложность для различения представляют случаи $A_{m} =p_{0}$ и $A_{m} =p_{1}$, так как в них распределения $g_{m}( \xi _{0})$ наиболее близки. Поэтому сперва рассмотрим проверку простой гипотезы $\hat{H}_{0} :A_{m} =p_{0}$, против простой альтернативы $\hat{H}_{1} :A_{m} =p_{1}$. В этом случае легко построить наиболее мощный критерий, как критерий отношения правдоподобий: 
\begin{equation}
\label{criteria1}
A_{m}^{t} \geqslant p_{0} +z_{\alpha }\sqrt{p_{0}( 1-p_{0}) /| O_{t}| },
\end{equation}
где $z_{\alpha }$ – квантиль уровня $\alpha$ стандартного нормального распределения. Для данного критерия можно вычислить объем тестовой выборки $| O_{t}| $ так, чтобы ошибка второго рода равнялась $\beta$:
\begin{equation}
\label{volume1}
| O_{t}| =\left(\frac{z_{\alpha }\sqrt{p_{0}( 1-p_{0})} +z_{\beta }\sqrt{p_{1}( 1-p_{1})}}{p_{0} -p_{1}}\right)^{2}.
\end{equation}
Так как критерий наиболее мощный, то такой объем тестовой выборки необходимый и достаточный для достижения ошибок первого и второго рода $\alpha$ и $\beta$. Заметим, что событие из критерия для модели с $A_{m}  >p_{0}$ будет происходить вероятнее, чем для модели с  $A_{m} =p_{0}$, и наоборот, для модели с $A_{m} < p_{1}$ это же событие будет происходить менее вероятно чем для модели с $A_{m} =p_{1}$, поэтому критерий при данном объеме тестовой выборки будет подходить под ограничения на ошибки первого и второго рода при проверке сложной гипотезы  $H_{0} :A_{m} \geqslant p_{0}$, против альтернативы $H_{1} :A_{m} \leqslant p_{1}$. 


```python
import scipy.stats as sps
import numpy as np
import math

def CalculatRequiredVolume(alpha, beta, p0, p1):
    if (alpha >= 0.5) or (beta >= 0.5):
        return -1
    a = sps.norm(loc=0, scale=1).ppf(alpha)
    b = sps.norm(loc=0, scale=1).ppf(beta)
    res = p1*(1-p1)
    res = res ** 0.5
    res *= b
    res += a * (p0*(1-p0)) ** 0.5
    res /= p0 - p1
    res *= res
    return res
```


```python
alpha = 0.05  # уровень ошибки первого рода 
beta = alpha  # уровень ошибки второго рода
p0 = 0.9987   # граница высококачественных моделей
p1 = 0.9979   # граница низкокачественных моделей

print("Чтобы различать модели с качеством <=", p1, "и модели с качеством >=", p0) 
print("с ошибками первого и второго рода на уровне", alpha, "и", beta, "соответственно")
print("необходим размер тестовой выборки:", math.ceil(CalculatRequiredVolume(alpha,beta,p0,p1)))
```

    Чтобы различать модели с качеством <= 0.9979 и модели с качеством >= 0.9987
    с ошибками первого и второго рода на уровне 0.05 и 0.05 соответственно
    необходим размер тестовой выборки: 28294


## Гипотеза о преимуществе
Допустим две модели $m_{1} ,\ m_{2}$ показывают разную точность $A_{m1}^{t}$ и $A_{m2}^{t} ,\ A_{m1}^{t}  >A_{m2}^{t}$, на тестовой выборке, в таком случае резонно ставить вопрос о соотношении между  $A_{m1}$ и $A_{m2}$. Рассмотрим статистику 
\begin{equation}
\label{stat1}
St\left( A_{m1}^{t} ,A_{m2}^{t}\right) =\sqrt{2| O_{t}| }\frac{A_{m1}^{t} -A_{m2}^{t}}{\sqrt{A_{m1}^{t} +A_{m2}^{t}}\sqrt{1-A_{m1}^{t} +1-A_{m2}^{t}}} ,
\end{equation}
которая асимптотически нормальная при $| O_{t}| \rightarrow \infty $ и $A_{m1} =A_{m2}$. Для проверки гипотезы $H_{0} :A_{m1} \leqslant A_{m2}$, против альтернативы $H_{1} :A_{m1}  >A_{m2}$ введем P-значение $p_{St}$:
\begin{equation}
\label{pvalue}
p_{St}\left( A_{m1}^{t} ,A_{m2}^{t}\right) =1-F_{\mathcal{N}}\left( St\left( A_{m1}^{t} ,A_{m2}^{t}\right)\right) ,
\end{equation}
где $F_{\mathcal{N}}$ -- функция стандартного нормального распределения. Тогда если отвергать гипотезу $H_{0}$ при $p_{St} \leqslant \alpha $ то ошибка первого рода также будет на уровне $\alpha$. Инвертировав выражение выше относительно $A_{m2}^{t}$, можно вычислить диапазон значений $A_{m2}^{t}$ при которых $p_{St} \leqslant \alpha $ при фиксированном $A_{m1}^{t}$:
\begin{equation}
\label{range}
A_{m2}^{t} \in \left[ 0,\frac{2| O_{t}| A_{m1}^{t} -z_{\alpha }^{2} A_{m1}^{t} +z_{\alpha }^{2} -\sqrt{D}}{2| O_{t}| +z_{\alpha }^{2}}\right] ,
\end{equation}
\begin{equation}
D=\left( 2| O_{t}| A_{m1}^{t} -z_{\alpha }^{2} A_{m1}^{t} +z_{\alpha }^{2}\right)^{2} -A_{m1}^{t}\left( 2| O_{t}| +z_{\alpha }^{2}\right)\left( 2| O_{t}| A_{m1}^{t} -2z_{\alpha }^{2} +z_{\alpha }^{2} A_{m1}^{t}\right) .
\end{equation}
Другими словами, если $A_{m2}^{t}$ принадлежит данному диапазону, то можно говорить, что преимущество в точности модели $m1$ над моделью $m2$ статистически значимо, на уровне ошибки первого рода $\alpha$.

Также, можно вычислить необходимый размер тестовой выборки $| O_{t}| $, при котором значения Accuracy  $A_{m1}^{t}$ и $A_{m2}^{t} ,\ A_{m1}^{t}  >A_{m2}^{t}$, говорят о статистически значимом преимуществе модели $m1$ над моделью $m2$: 
\begin{equation}
\label{volume2}
| O_{t}| \geqslant z_{\alpha }^{2}\frac{\left( A_{m1}^{t} +A_{m2}^{t}\right)\left( 1-A_{m1}^{t} +1-A_{m2}^{t}\right)}{2\left( A_{m1}^{t} -A_{m2}^{t}\right)^{2}} .
\end{equation}
Эта формула дает представление о том, каким должен быть размер тестовой выборки, чтобы по полученным $A_{m1}^{t}$ и $A_{m2}^{t} ,\ A_{m1}^{t}  >A_{m2}^{t}$ можно было бы заявлять о преимуществе модели $m1$ над моделью $m2$.


```python
import scipy.stats as sps
import numpy as np
import math

def BorderDistinctness(alpha, A, n):
    z = sps.norm(loc=0, scale=1).ppf(alpha)
    D = 2*n*A -z*z*A + z*z
    D *= D
    D -= (2*n + z*z) * (2*n*A*A - 2*z*z*A + z*z*A*A)
    res = 2*n*A - z*z*A + z*z - D**(0.5)
    res /= 2*n +z*z
    return res

def RequiredSizeDistinguishability(alpha, A1, A2):
    if A1 <= A2:
        return -1
    z = sps.norm(loc=0, scale=1).ppf(alpha)
    res = A1 + A2
    res *= 2 - A1 - A2
    res /= 2 * (A1 - A2) ** 2
    res *= z * z
    return res
```


```python
alpha = 0.05  # уровень ошибки первого рода 
A1 = 0.9395   # Accuracy рассматриваемой модели
n = 10000     # объем тестовой выборки датасета

print("Модель с Accuracy", A1, "на тестовой выборке с объемом", n) 
print("имеет статистически значимое преимущество с уровнем ошибки первого рода", alpha)
print("над моделями с Accuracy <=", BorderDistinctness(alpha1, A1, n1))
```

    Модель с Accuracy 0.9395 на тестовой выборке с объемом 10000
    имеет статистически значимое преимущество с уровнем ошибки первого рода 0.05
    над моделями с Accuracy <= 0.9338343554205788



```python
alpha = 0.05  # уровень ошибки первого рода
A1 = 0.9987   # Accuracy более точной из двух моделей
A2 = 0.9984   # Accuracy менее точной из двух моделей

print("Чтобы модель с Accuracy", A1, "имела статистически значимое")
print("преимущество над моделью с Accuracy", A2) 
print("с уровнем ошибки первого рода", alpha)
print("необходимо, чтобы объем тестовой выборки был >=", math.ceil(RequiredSizeDistinguishability(alpha1, A1, A2)))
```

    Чтобы модель с Accuracy 0.9987 имела статистически значимое
    преимущество над моделью с Accuracy 0.9984
    с уровнем ошибки первого рода 0.05
    необходимо, чтобы объем тестовой выборки был >= 87053
