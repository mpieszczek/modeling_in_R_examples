Potrzebne biblioteki
====================

``` r
library(cluster.datasets)
library(tidyverse)
library(tidymodels)
library("cluster")
library("factoextra")
```

Zbiór danych
============

### Uzyjemy zbioru danych mleka różnych ssaków.

Ładowanie i wyswietlanie zbiór danych `all.mammals.milk.1956`. Dane
zawierają wartości procentowe. Np mleko końskie ma 90.1% wody oraz 1%
tłuszczu.

``` r
data(all.mammals.milk.1956)
all.mammals.milk.1956
```

| name       |  water|  protein|   fat|  lactose|   ash|
|:-----------|------:|--------:|-----:|--------:|-----:|
| Horse      |   90.1|      2.6|   1.0|      6.9|  0.35|
| Orangutan  |   88.5|      1.4|   3.5|      6.0|  0.24|
| Monkey     |   88.4|      2.2|   2.7|      6.4|  0.18|
| Donkey     |   90.3|      1.7|   1.4|      6.2|  0.40|
| Hippo      |   90.4|      0.6|   4.5|      4.4|  0.10|
| Camel      |   87.7|      3.5|   3.4|      4.8|  0.71|
| Bison      |   86.9|      4.8|   1.7|      5.7|  0.90|
| Buffalo    |   82.1|      5.9|   7.9|      4.7|  0.78|
| Guinea Pig |   81.9|      7.4|   7.2|      2.7|  0.85|
| Cat        |   81.6|     10.1|   6.3|      4.4|  0.75|
| Fox        |   81.6|      6.6|   5.9|      4.9|  0.93|
| Llama      |   86.5|      3.9|   3.2|      5.6|  0.80|
| Mule       |   90.0|      2.0|   1.8|      5.5|  0.47|
| Pig        |   82.8|      7.1|   5.1|      3.7|  1.10|
| Zebra      |   86.2|      3.0|   4.8|      5.3|  0.70|
| Sheep      |   82.0|      5.6|   6.4|      4.7|  0.91|
| Dog        |   76.3|      9.3|   9.5|      3.0|  1.20|
| Elephant   |   70.7|      3.6|  17.6|      5.6|  0.63|
| Rabbit     |   71.3|     12.3|  13.1|      1.9|  2.30|
| Rat        |   72.5|      9.2|  12.6|      3.3|  1.40|
| Deer       |   65.9|     10.4|  19.7|      2.6|  1.40|
| Reindeer   |   64.8|     10.7|  20.3|      2.5|  1.40|
| Whale      |   64.8|     11.1|  21.2|      1.6|  0.85|
| Seal       |   46.4|      9.7|  42.0|      0.0|  0.85|
| Dolphin    |   44.9|     10.6|  34.9|      0.9|  0.53|

### Dodajmy do ramki danych poniższe dane mleka ludzkiego.

``` r
Man <- data.frame(
  name = "Man",
  water = 88.1,
  protein = 0.85,
  fat = 0.4,
  lactose = 7.0,
  ash = 0.2
)
mammals <- rbind(Man, all.mammals.milk.1956)
```

### Ze względu na grupowanie i łatwiejszy odczyt późniejszych wyników użyjemy kolumny `name` jako klucza głównego.

``` r
ssaki <- mammals
rownames(ssaki) <- ssaki$name
ssaki <- ssaki %>% select(-name)
ssaki %>% head(5)
```

|           |  water|  protein|  fat|  lactose|   ash|
|:----------|------:|--------:|----:|--------:|-----:|
| Man       |   88.1|     0.85|  0.4|      7.0|  0.20|
| Horse     |   90.1|     2.60|  1.0|      6.9|  0.35|
| Orangutan |   88.5|     1.40|  3.5|      6.0|  0.24|
| Monkey    |   88.4|     2.20|  2.7|      6.4|  0.18|
| Donkey    |   90.3|     1.70|  1.4|      6.2|  0.40|

### Wstępne przeskalowanie danych:

``` r
ssaki_scaled <- ssaki %>% scale()
head(ssaki_scaled)
```

    ##               water    protein        fat   lactose       ash
    ## Man       0.7502485 -1.3822256 -0.9084438 1.4661653 -1.259537
    ## Horse     0.9076221 -0.9130629 -0.8512306 1.4129989 -0.947255
    ## Orangutan 0.7817232 -1.2347745 -0.6128420 0.9345015 -1.176262
    ## Monkey    0.7738546 -1.0203001 -0.6891263 1.1471670 -1.301174
    ## Donkey    0.9233595 -1.1543466 -0.8130884 1.0408342 -0.843161
    ## Hippo     0.9312282 -1.4492489 -0.5174866 0.0838393 -1.467725

Grupowanie metodą *k*-średnich
==============================

### Wyznaczamy podział na 3 klastry z pomocą algorytmy k-średnich.

``` r
set.seed(1234)
klastry_k_means <- ssaki_scaled %>%
  kmeans(centers = 3, iter.max = 100, nstart = 25)
```

Czy nasz podział ma sens?
-------------------------

Najmniejszy klaster to nr 3 jest w nim tylko foka i delfin. Widać że
proporcje się u nich znacząco różnią. Posiadają bardzo mało wody i dużo
tłuszczu. Jeżeli spojrzymy na oryginalne dane to jest to poniżej 50%
wody i powyżej 30% tłuszczu. Różnice między klastrem 1 i 2 są bardziej
subtelnę to ciężko ocenić. Na pewno 1 posiadają więcej białka oraz
posiadają średnio mniej wody i laktozy.

``` r
ssaki_scaled_z_klastrem <- klastry_k_means %>%
  augment(ssaki_scaled)

ssaki_scaled_z_klastrem %>%
  count(.cluster)
```

| .cluster |    n|
|:---------|----:|
| 1        |    9|
| 2        |   15|
| 3        |    2|

``` r
ssaki_scaled_z_klastrem %>%
  filter(.cluster == 1)
```

| .rownames  |       water|    protein|         fat|     lactose|         ash| .cluster |
|:-----------|-----------:|----------:|-----------:|-----------:|-----------:|:---------|
| Guinea Pig |   0.2623903|  0.3737835|  -0.2600269|  -0.8199893|   0.0936846| 1        |
| Cat        |   0.2387842|  1.0976346|  -0.3458468|   0.0838393|  -0.1145033| 1        |
| Pig        |   0.3332084|  0.2933556|  -0.4602733|  -0.2883254|   0.6141543| 1        |
| Dog        |  -0.1782559|  0.8831602|  -0.0407094|  -0.6604901|   0.8223422| 1        |
| Rabbit     |  -0.5716900|  1.6874392|   0.3025701|  -1.2453203|   3.1124092| 1        |
| Rat        |  -0.4772658|  0.8563509|   0.2548924|  -0.5009909|   1.2387180| 1        |
| Deer       |  -0.9965988|  1.1780625|   0.9319159|  -0.8731556|   1.2387180| 1        |
| Reindeer   |  -1.0831543|  1.2584904|   0.9891292|  -0.9263220|   1.2387180| 1        |
| Whale      |  -1.0831543|  1.3657276|   1.0749491|  -1.4048195|   0.0936846| 1        |

``` r
ssaki_scaled_z_klastrem %>%
  filter(.cluster == 2)
```

| .rownames |       water|     protein|         fat|    lactose|         ash| .cluster |
|:----------|-----------:|-----------:|-----------:|----------:|-----------:|:---------|
| Man       |   0.7502485|  -1.3822256|  -0.9084438|  1.4661653|  -1.2595368| 2        |
| Horse     |   0.9076221|  -0.9130629|  -0.8512306|  1.4129989|  -0.9472550| 2        |
| Orangutan |   0.7817232|  -1.2347745|  -0.6128420|  0.9345015|  -1.1762617| 2        |
| Monkey    |   0.7738546|  -1.0203001|  -0.6891263|  1.1471670|  -1.3011744| 2        |
| Donkey    |   0.9233595|  -1.1543466|  -0.8130884|  1.0408342|  -0.8431610| 2        |
| Hippo     |   0.9312282|  -1.4492489|  -0.5174866|  0.0838393|  -1.4677247| 2        |
| Camel     |   0.7187738|  -0.6717792|  -0.6223775|  0.2965048|  -0.1977785| 2        |
| Bison     |   0.6558243|  -0.3232583|  -0.7844818|  0.7750023|   0.1977785| 2        |
| Buffalo   |   0.2781276|  -0.0283560|  -0.1932781|  0.2433385|  -0.0520470| 2        |
| Fox       |   0.2387842|   0.1593091|  -0.3839890|  0.3496712|   0.2602349| 2        |
| Llama     |   0.6243496|  -0.5645420|  -0.6414486|  0.7218359|  -0.0104094| 2        |
| Mule      |   0.8997535|  -1.0739187|  -0.7749462|  0.6686695|  -0.6974295| 2        |
| Zebra     |   0.6007436|  -0.8058257|  -0.4888799|  0.5623368|  -0.2185973| 2        |
| Sheep     |   0.2702589|  -0.1087839|  -0.3363113|  0.2433385|   0.2185973| 2        |
| Elephant  |  -0.6189021|  -0.6449699|   0.7316695|  0.7218359|  -0.3643288| 2        |

``` r
ssaki_scaled_z_klastrem %>%
  filter(.cluster == 3)
```

| .rownames |      water|    protein|       fat|    lactose|         ash| .cluster |
|:----------|----------:|----------:|---------:|----------:|-----------:|:---------|
| Seal      |  -2.530992|  0.9903974|  3.058342|  -2.255482|   0.0936846| 3        |
| Dolphin   |  -2.649022|  1.2316811|  2.381318|  -1.776984|  -0.5725167| 3        |

``` r
mammals %>%
  filter(name == "Dolphin" | name == "Seal")
```

| name    |  water|  protein|   fat|  lactose|   ash|
|:--------|------:|--------:|-----:|--------:|-----:|
| Seal    |   46.4|      9.7|  42.0|      0.0|  0.85|
| Dolphin |   44.9|     10.6|  34.9|      0.9|  0.53|

Grupowanie hierarchiczne
========================

### Zastosujemy teraz metodę klastrowania hierarchicznego z łączeniami *complete*, *average* i *single*.

``` r
# maciez odległosci
res.dist <- dist(ssaki_scaled, method = "euclidean")
as.matrix(res.dist)[1:6, 1:6]
```

    ##                 Man     Horse Orangutan    Monkey    Donkey     Hippo
    ## Man       0.0000000 0.5903390 0.6322295 0.5321101 0.6672813 1.4643295
    ## Horse     0.5903390 0.0000000 0.6764480 0.5015944 0.4574521 1.5610868
    ## Orangutan 0.6322295 0.6764480 0.0000000 0.3357236 0.4346157 0.9412866
    ## Monkey    0.5321101 0.5015944 0.3357236 0.0000000 0.5260891 1.1817904
    ## Donkey    0.6672813 0.4574521 0.4346157 0.5260891 0.0000000 1.2166877
    ## Hippo     1.4643295 1.5610868 0.9412866 1.1817904 1.2166877 0.0000000

``` r
model_complete <- hclust(d = res.dist, method = "complete")
model_average <- hclust(d = res.dist, method = "average")
model_single <- hclust(d = res.dist, method = "single")
```

``` r
fviz_dend(model_complete,
  k = 3, # liczba grup
  cex = 0.5, # rozmiar etykiet
  # k_colors = c("#2E9FDF", "#00AFBB", "#E7B800",),
  color_labels_by_k = TRUE,
  rect = TRUE,
  main = "Complete"
)
```

![](Cluster_kmeans_hirarchy_files/figure-markdown_github/unnamed-chunk-9-1.png)

``` r
fviz_dend(model_average,
  k = 3, # liczba grup
  cex = 0.5, # rozmiar etykiet
  # k_colors = c("#2E9FDF", "#00AFBB", "#E7B800",),
  color_labels_by_k = TRUE,
  rect = TRUE,
  main = "Average"
)
```

![](Cluster_kmeans_hirarchy_files/figure-markdown_github/unnamed-chunk-9-2.png)

``` r
fviz_dend(model_single,
  k = 3, # liczba grup
  cex = 0.5, # rozmiar etykiet
  # k_colors = c("#2E9FDF", "#00AFBB", "#E7B800",),
  color_labels_by_k = TRUE,
  rect = TRUE,
  main = "Single"
)
```

![](Cluster_kmeans_hirarchy_files/figure-markdown_github/unnamed-chunk-9-3.png)

Wnioski
=======

Jak widać grupa nr 3 się ostała jest na tyle charakterystyczna że jest
wykrywana w każdym przypadku. O pozostałych ciężko powiedzieć to samo.
Jeżeli zadamy sobie pytanie: Do kogo w tym zbiorze danych najbardziej
podobny jest człowiek? To najłatiwej chyba spojrzeć na macierz
odległości, z użyciem metryki euklidesowej. Najbardziej podobnym do
człowieka(poza samym człowiekiem) na tym zbiorze danych jest jest małpa.

``` r
as.matrix(res.dist)[1, ] %>% sort()
```

    ##        Man     Monkey      Horse  Orangutan     Donkey       Mule      Hippo 
    ##  0.0000000  0.5321101  0.5903390  0.6322295  0.6672813  1.0426844  1.4643295 
    ##      Zebra      Llama      Camel      Bison    Buffalo      Sheep   Elephant 
    ##  1.5591933  1.6941337  1.7558460  1.9357605  2.3496086  2.4206540  2.5423443 
    ##        Fox        Pig        Cat Guinea Pig        Dog        Rat       Deer 
    ##  2.5434707  3.1259091  3.1543216  3.2862941  3.9501823  4.2405599  4.9706312 
    ##      Whale   Reindeer     Rabbit    Dolphin       Seal 
    ##  4.9920158  5.0892832  6.2532379  6.3403548  6.9147040

### Podobnie możemy odczytać, że 2 najbardziej do siebie zblizone gatunki to sarna i renifer.

``` r
min_dist <- min(res.dist) # res.dist nie ma przękątnych więc minimum bedize poprzwne
distance_matrix <- as.matrix(res.dist) # mapujemy do macierzy żeby móć uzyć funkcji which, res.dist jest klasy dist która nie ma wielu funkcji szukania
which(distance_matrix == min_dist, arr.ind = TRUE)
```

    ##          row col
    ## Reindeer  23  22
    ## Deer      22  23

``` r
min_dist
```

    ## [1] 0.1416352

### Z dendrogramu można wyczytać dosyć nieintuicyjne podobieństwa składu mleka.

Tak jak podejrzewamy podobieństwo foka jest podobna do delfina(ssaki
wodne). Zaskakują gatunki podobne do człowieka (gałąź po lewej), poza
małpą i orangutanem pojawiają się tam o dziwo: hipopotam, koń, muł i
osioł. W szczególności koń który jest prawie tak samo podobny do
człowieka jak małpa.

``` r
fviz_dend(model_complete,
  k = 3, # liczba grup
  cex = 0.5, # rozmiar etykiet
  # k_colors = c("#2E9FDF", "#00AFBB", "#E7B800",),
  color_labels_by_k = TRUE,
  rect = TRUE,
  main = "Complete"
)
```

![](Cluster_kmeans_hirarchy_files/figure-markdown_github/unnamed-chunk-12-1.png)
