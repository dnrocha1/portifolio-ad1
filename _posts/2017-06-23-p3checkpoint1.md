---
layout: post
title: "AD1 2017.1, Problema 3 - Checkpoint 1"
date: "2017-06-23"
author: "Daniyel Rocha"
published: true
tags: [htmlwidgets, r]
#output: html_document
---



Importando as bibliotecas necessárias.

{% highlight r %}
library(tidyverse, warn.conflicts = F)
library(rvest, warn.conflicts = F)
library(plotly, warn.conflicts = F)
library(cluster)
library(ggdendro)
source("https://raw.githubusercontent.com/nazareno/ciencia-de-dados-1/master/3-Agrupamento-e-PCA/plota_solucoes_hclust.R")
theme_set(theme_light())
{% endhighlight %}

Inicialmente iremos coletar os dados brutos de Steve Carell no site Rotten Tomatoes para a análise.


{% highlight r %}
from_page <- read_html("https://www.rottentomatoes.com/celebrity/steve_carell") %>% 
    html_node("#filmographyTbl") %>% 
    html_table(fill=TRUE) %>%
    as.tibble()
{% endhighlight %}

Precisamos filtrar dados e variáveis dessa tabela para que fiquem mais organizados e o trabalho seja facilitado. Observando os nomes das variáveis, temos:


{% highlight r %}
filmes <- from_page %>%
  filter(RATING != "No Score Yet") %>%
  mutate(RATING = as.numeric(gsub("%", "", RATING)),
         `BOX OFFICE` = as.numeric(gsub("[$|M]", "", `BOX OFFICE`)),
         CREDIT = as.character(gsub("Screenwriter|Executive|Producer", "", CREDIT))) %>%
  na.omit()
{% endhighlight %}



{% highlight text %}
## Warning in evalq(as.numeric(gsub("[$|M]", "", `BOX OFFICE`)),
## <environment>): NAs introduzidos por coerção
{% endhighlight %}



{% highlight r %}
names(filmes)
{% endhighlight %}



{% highlight text %}
## [1] "RATING"     "TITLE"      "CREDIT"     "BOX OFFICE" "YEAR"
{% endhighlight %}

***
Nossas variáveis sobre os filmes de Steve Carell são:

* RATING: Percentual de avaliação no Rotten Tomatoes
* TITLE: Título do filme
* CREDIT: Personagem no filme
* BOX OFFICE: Ganhos de bilheteria (em milhões)
* YEAR: Ano de lançamento

Os dados foram filtrados de modo que as avaliações não aparecessem com as porcentagens e os valores de bilheteria fossem somente numéricos. Tratamos também os créditos do ator, para que nossa analise só leve em conta as atuações do mesmo - não considerando trabalhos onde ele possa ter tido outra função. Essa filtragem facilita a manipulação e organização dos dados para quando formos trabalhar com os mesmos.

***
Pesos iguais para a escala faz com que os dados sejam representados de forma igualitária, mas desejo observar os dados de outra maneira, de modo que filmes com arrecadação maior não tenham a mesma representatividade de filmes com pouca arrecadação. Isso irá facilitar a observação de grupamentos dos dados, já que não observaremos os dados em uma escala que trata todos os valores de uma mesma forma, dando um peso maior para certos valores e nos revelando grupamentos antes não vistos.

Após diversas experimentações de agrupamentos dos nossos dados, foi feita a opção por escolher por escolher grupamentos de duas dimensões. O método de cálculo da distância foi o euclidiano e os grupos são formados utilizando o centroide.  
Realizamos modificação na representação da escala, já que pesos iguais para a escala faz com que os dados sejam representados de forma igualitária, mas optei por observar os dados de outra maneira, utilizando a escala logarítmica, de modo que filmes com arrecadação maior não tenham a mesma representatividade de filmes com pouca arrecadação. Isso irá facilitar a observação de grupamentos dos dados, já que não observaremos os dados em uma escala que trata todos os valores de uma mesma forma, dando um peso maior para certos valores e nos revelando grupamentos antes não vistos.


{% highlight r %}
agrupamento_h_2d = filmes %>% 
    column_to_rownames("TITLE") %>%
    select(RATING, `BOX OFFICE`) %>% 
    mutate(`BOX OFFICE` = log10(`BOX OFFICE`)) %>% 
    mutate_all(funs(scale)) %>% 
    dist(method = "euclidean") %>% 
    hclust(method = "centroid")
{% endhighlight %}



{% highlight text %}
## Warning: Setting row names on a tibble is deprecated.
{% endhighlight %}



{% highlight r %}
atribuicoes <- tibble(k = 1:6) %>% 
        group_by(k) %>% 
        do(cbind(filmes, 
                 grupo = as.character(cutree(agrupamento_h_2d, .$k)))) 
{% endhighlight %}



{% highlight text %}
## Warning in bind_rows_(x, .id): Unequal factor levels: coercing to
## character
{% endhighlight %}



{% highlight text %}
## Warning in bind_rows_(x, .id): binding character and factor vector,
## coercing into character vector
{% endhighlight %}



{% highlight text %}
## Warning in bind_rows_(x, .id): binding character and factor vector,
## coercing into character vector
{% endhighlight %}



{% highlight text %}
## Warning in bind_rows_(x, .id): binding character and factor vector,
## coercing into character vector
{% endhighlight %}



{% highlight text %}
## Warning in bind_rows_(x, .id): binding character and factor vector,
## coercing into character vector
{% endhighlight %}



{% highlight text %}
## Warning in bind_rows_(x, .id): binding character and factor vector,
## coercing into character vector
{% endhighlight %}



{% highlight text %}
## Warning in bind_rows_(x, .id): binding character and factor vector,
## coercing into character vector
{% endhighlight %}



{% highlight r %}
p <- atribuicoes %>% 
        ggplot(aes_string(x = "RATING", y = "`BOX OFFICE`", colour = "grupo")) + 
        geom_jitter(width = .02, height = 0, size = 2, alpha = .6,
                    aes(text = paste(
    "Grupo:", grupo, "<br>",
    "Título:", TITLE, "<br>",
    "Avaliação:", RATING, "<br>",
    "Receita:", `BOX OFFICE`, "<br>",
    "Lançamento:", YEAR)
    )) + 
        facet_wrap(~ paste(k, " grupos")) + 
        xlab("RATING") + scale_y_log10()
{% endhighlight %}



{% highlight text %}
## Warning: Ignoring unknown aesthetics: text
{% endhighlight %}



{% highlight r %}
ggplotly(p,tooltip = c("text")) %>%
  layout(autosize = T, width = 1000, height = 950, margin = list(l = 50, r = 150, b = 100, t = 100, pad = 4))
{% endhighlight %}



{% highlight text %}
## Warning: Specifying width/height in layout() is now deprecated.
## Please specify in ggplotly() or plot_ly()
{% endhighlight %}

![plot of chunk unnamed-chunk-4](/portifolio-ad1/figure/source/P3CP1/2017-06-23-p3checkpoint1/unnamed-chunk-4-1.png)

Observando os resultados desses agrupamentos, vemos que uma boa maneira de observar grupos relativamente bem definidos é com 4 grupos. Isso pode ser justificado já que para essa quantidade de grupos há uma boa atribuição de cada variável no seu respectivo grupo, para boa parte dos dados. No geral, a distancia média de um ponto para os outros pontos de seu grupo é parecida com a distância média entre esse ponto e os demais grupos. Para uma maior ou menor quantidade de grupamentos, talvez eles não estariam alocados no seu grupo de uma maneira tão adequada quanto essa quantidade de 4 grupos.

***

Podemos definir os grupos como sendo:

* Grupo 1: "Sucessos", são filmes que tiveram certo sucesso de bilheteria e de crítica, estando bem em pelo menos um desses quesitos. Ex: "Despicable Me 2", que não é um filme razoável na avaliação, entretanto foi seu filme com maior bilheteria.
* Grupo 2: "Pastelão", os filmes desse grupo geralmente são aqueles que conseguem ir bem no cinema graças a propaganda massiva feita para eles, mas decepcionam a todos pela qualidade. Ex: "Evan Almighty" (A Volta do Todo Poderoso), que pegou carona no sucesso do antecessor, mas que não teve a mesma qualidade da versão original.
* Grupo 3: "Alternativos", essa categoria de filmes não agrada a todos, mas aparentemente oferece alternativas que podem surpreender pela qualidade - apesar de não serem sucesso de bilheteria. Ex: "Foxcatcher", que concorreu a diversas estatuetas do Oscar.
* Grupo 4: "Fracassos", esse grupo unário foi o famoso ruim de público e de crítica. Ex: "Sleepover".
