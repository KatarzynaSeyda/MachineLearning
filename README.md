# Analiza notowań sektorów na giełdzie  📈

https://colab.research.google.com/drive/1VRk2ZH_AyqvYOatS1U6PgXedCnWYmvKF?authuser=1#scrollTo=9Xmt7Q-B1x5g

Program pozwala na analizowanie danych dotyczących notowań najwięszych spółek giełdowych per sektor. Dane zawierają codzienne notowania 61 największych spółek pogrupowanych w 20 sektorów. 📶

🔍 Użytkownik może wybrać sektor, który chce przeanalizować.  
📆 Analizie poddawane jest ostatnie 104 tygodnie (~2 lata).

### ________ 🔷 Przygotowanie danych🔷 ____________________
⚙️**Indeksy sektorowe** przedstawiają średnią tygodniową z indeksów dziennych. Te natomiast są średnią cen zamknięcia spółek w danym sektorze ważoną wolumenem sprzedaży.

W pierwszym kroku analizy program przygotowuje:
+ **statystyki opisowe** zbioru danych,
+ **mapę korelacji** między indeksami sektorowymi,
+ wykres liniowy pokazujący **sentyment na rynku**. 


Program opiera prognozy o model 🚀**XGBoost**🚀. W związku z tym, że dane zawierają szereg czasowy, stworzono **zmienne opóźnione** 🕔 (z maksymalnym opóźnieniem 5 tygodni). Ze względu na analizowany temat uznano, że 5 tygodniowe indeksy sektorowe są wystarczające do prognozy aktualnego tygodnia.
Wiersze z brakami danych przez stworzenie danych opóźnionych są usuwane (są to najstarsze indeksy, dla których nie można już stworzyć indeksu opóźnionego).


### ________ 🔷 Selekcja zmiennych 🔷 ____________________
Następnie metodą **Group Lasso** 𓍯𓂃 program wybiera istotne zmienne do modelu. Zmienne opóźnione grupowane są według sektora do którego należą.
W metodzie tej zmienne X są standaryzowane metodą MinMaxScaler.
Lista wyselekcjonowanych zmiennych (lub grup zmiennych) jest dalej przekazana do treningu modelu XGBoost.

W kolejnym kroku zbudowano model XGBoost na wyselekcjonowanych zmiennych i obliczono **Feature Importance**📊 dla każdej z nich.
Oceniająć globalny wpływ zmiennych na zmienną objaśnianą program eliminuje najmniej istotne zmienne. W modelu finalnym pozostają zmienne wyjaśniające w 95% zmienną prognozowaną.

Po wyselekcjonowaniu zmiennych do modelu program optymalizuje ostateczny model XGBoost.


### ________ 🔷 Modelowanie 🔷 ____________________
Hiperparametry modelu XGBoost są optymalizowane modułem **Optuna** 🔄:
+ _learning_rate_: szybkość uczenia, wpływa na długość kroku iteracji
+ _reg_alpha_: redukcja wag cech nieistotnych, pomaga w selekcji cech 
+ _reg_lambda_: ograniczenie wielkości wag, im lambda większa tym Similarity Score mniejszy, model się uogólnia

Pozostałe hiperparametry ustawione w modelu XGBoost:
+ _reg:squarederror_: funkcja kosztu dla regresji 
+ _eval_metric_: RMSE do oceny modelu
+ _max_depth_=6: głębokość drzewa decyzyjnego
+ _gamma_=1: minimalna redukcja kosztu, wymagana do podziału drzewa

Podczas optymalizacji hiperparametrów wykorzystane jest przesuwające się okno czasowe (zakres danych treningowych i testowych przesuwa się o 1 tydzień).
Dla każdego zestawu hiperparametrów wykonywany jest trening modelu w każdym oknie czasowym. Wyniki są agregowane jako średnia RMSE.
Po ustaleniu optymalnych hiperparametrów modelu następuje ostateczny trening modelu.


### ________ 🔷 Ocena modelu 🔷 ____________________
🧮 Wyniki modelu oceniane są według **RMSE** (Root Mean Squared Error) oraz **MAPE** (Mean Absolute Percentage Error).

Program pozwala również na wizualizację wpływu zmiennych na wartość analizowanego sektora (**Shapley values**).


### ________ 🔷 Prognoza dla użytkownika 🔷 ____________________
Ostatcznie program dokonuje **prognozę**🎯 wartości indeksu sektora na kolejny tydzień.
Na podstawie wartości prognozowanej dokonywana jest rekomendacja KUP/SPRZEDAJ.


