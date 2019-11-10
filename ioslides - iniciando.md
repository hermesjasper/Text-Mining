---
title: 'Mineração e análise de textos no <img src="Rstudio_logo.png" height=90/>'
date: "8 de novembro de 2018"
output:
  ioslides_presentation:
    logo: UnB_logo.png
    smaller: TRUE
    widescreen: TRUE
    transition: slower
    theme: "lumen"
    css: Estilos.css
---

```{r, include = FALSE}
setwd(getwd())
knitr::opts_chunk$set(echo = FALSE)
```

```{r, echo = FALSE, include = FALSE, eval = TRUE, message = FALSE}
library(pacman)

pacman::p_load(devtools, rtweet, tm, RColorBrewer, cluster, fpc, httpuv, SnowballC,
              ggplot2, wordcloud, wordcloud2, tidytext,
              stringr, tidyverse, knitr, png, webshot, htmlwidgets)

devtools::install_github("lchiffon/wordcloud2")
```

# Mineração de texto: Dando os primeiros passos {data-background=text-mining-image.jpg  data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}

## Pacotes {.build}
A mineração de texto é um recurso utilizado por cientistas de dados para coletar dados em forma de texto, visando obter informações relevantes.

> É comum que esse texto seja minerado de redes sociais ou de sites abertos a comentários, como sites de notícias, jornais, sites de streaming de vídeos como o YouTube, e alguns outros.

> Devido ao fato de alguns sites serem minerados com frequência, existem alguns pacotes que podem facilitar ao minerar dados de alguns sites. Por exemplo, nós temos pacotes no R para minerar dados do Twitter, do Facebook e do Instagram.

```{r, eval = FALSE, echo = TRUE}
library(rtweet)
library(Rfacebook)
library(instaR)
```

## Extraindo dados do Twitter
O pacote <span style = "font-family:Courier New">rTweet</span> foi construído para extrair dados do Twitter, e uma função interessante dele é <span style = "font-family:Courier New">search_tweets</span>, que procura todos os tweets que seguem os padrões que definimos.
```{r, echo = TRUE, eval=FALSE}
meu_dataFrame <- search_tweets(q, n = 100, type = "recent", include_rts = TRUE,
  geocode = NULL, max_id = NULL, parse = TRUE, token = NULL,
  retryonratelimit = FALSE, verbose = TRUE, lang = "...")
```

Isso facilita muito o trabalho de mineração. Apesar disso, esses casos são as exceções, não a regra, e não podemos esperar que sempre teremos paocotes para nos ajudar. Então... e se não tivermos?


## Método "força-bruta": Extraindo por HTML
Todas as páginas na rede possuem um código-fonte que pode ser acessado, e elas são sempre marcadas por HTML. A ideia de extrair dados em texto a partir do código-fonte é usar os padrões do HTML para extrair o código de interesse.

Para isso, devemos utilizar o R para entrar no código-fonte da página, e criar funções para que ele vá atrás desses padrões.


# Pré-processamento {data-background=text-mining-image.jpg  data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}

## Convertendo em corpus {.build}
```{r, eval = FALSE, echo = TRUE}
library(tm)
meuCorpus <- Corpus(VectorSource(meu_dataFrame)) # Criando um corpus a partir do dataframe
```
<h2>Tokenização</h2>
*Tokenizar* um texto é fundamentalmente dividir ele em unidades menores que possam ser mais facilmente analisadas computacionalmente. Geralmente, os seguintes pacotes são o suficiente:
```{r, eval = FALSE, echo = TRUE}
library(tidyverse)
library(tidytext)
library(glue)
library(data.table)
```

## Maiúsculas e minúsculas, pontuação e stopwords {.build}
As chamadas stopwords são palavras comuns da linguagem que, geralmente, não dizem muito sobre o texto, apesar de predominarem em qualquer sentença. Pra que possamos extrair informações relevantes, é importante que, antes limpemos essas palavras. Para isso, podemos usar funções do pacote <span style = "font-family:Courier New">tm</span>
```{r, eval = FALSE, echo = TRUE}
meuCorpus <- tm_map(meuCorpus, tolower) # Tornando todas as letras minúsculas
meuCorpus <- tm_map(meuCorpus, removePunctuation) # Removendo a pontuação
meuCorpus <- tm_map(meuCorpus, removeWords, stopwords("pt")) # Removendo as stopwords
meuCorpus <- tm_map(meuCorpus, removeNumbers) # Removendo os números
```

```{r, echo = FALSE}
#dadosTwitter <- VCorpus(VectorSource(dadosTwitter$texto))
#dadosTwitter <- tm_map(dadosTwitter, removeWords, c(stopwords("pt"), "acho","aqui","bolsonaro","cê","dar","dia","entao","entrar","faz","fazer","fica","ficar","gente","indo","mim","nada","nao","nessa","pois","porque","pra","pro","quer","queria","quero","quis","sair","sao","sei","ser","sim","tá","tava","ter","tô","toda","tudo","vai","vcs","vem","ver","voce","vou"))
```

Após a limpeza dos dados, podemos avaliar o que os dados têm a dizer. Vamos lá!

# Nuvens de palavras: Explorando... {data-background=text-mining-image.jpg  data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}
## Wordclouds
Uma prática útil (e divertida!) é criar *wordclouds* (nuvens de palavras), que fornecem uma boa visualização dos termos que mais frequentes. Os pacotes <span style = "font-family:Courier New">wordcloud</span> e <span style = "font-family:Courier New">wordcloud2</span> são apropriados pra isso. Esses gráficos ordenam as palavras pela frequência com que aparecem nos dados.
```{r, eval = FALSE, echo = TRUE}
#Wordcloud
wordcloud(words,freq,scale=c(4,.5),min.freq=3,max.words=Inf,
	random.order=TRUE, random.color=FALSE, rot.per=.1,
	colors="black",ordered.colors=FALSE)

#Wordcloud2
wordcloud2(data, size = 1, minSize = 0, gridSize =  0,
    fontFamily = 'Segoe UI', fontWeight = 'bold',
    color = 'random-dark', backgroundColor = "white",
    minRotation = -pi/4, maxRotation = pi/4, shuffle = TRUE,
    rotateRatio = 0.4, shape = 'circle', ellipticity = 0.65,
    widgetsize = NULL, figPath = NULL, hoverFunction = NULL)

```

Podemos fazer um exemplo bem simples, rodando <span style = "font-family:Courier New">wordcloud(demoFreq\$word, demoFreq\$freq)</span> e <span style = "font-family:Courier New">wordcloud2(demoFreq)</span> no console, e então comparar os dois pacotes.

## Wordclouds: Comparando os pacotes {.build}
```{r fig.align="left", eval = TRUE, warning = FALSE, echo = TRUE}
wordcloud(words = demoFreq$word, freq = demoFreq$freq, ordered.colors = TRUE)
```

## Wordclouds: Comparando os pacotes {.build}
```{r, fig.align="center", eval = TRUE, echo = TRUE}
wordcloud2(demoFreq, backgroundColor= "transparent", size = 0.9)
```


## Wordclouds: Dando forma aos gráficos
<div class="columns-2">
  <!-- ![wordcloudtwitterexample](wordcloudtwitterexample.png) -->
  <img src="wordcloudtwitterexample.png" height=444 width=500/ >
  

E que tal se colocássemos ele na forma da logo do Twitter?
```{r, echo = FALSE, eval = FALSE}
wordcloud2(demoFreq,
    figPath = system.file("examples/t.png", package = "wordcloud2"),
    color = "skyblue")
```

```{r, echo = TRUE, eval = FALSE}
wordcloud2(demoFreq,
    figPath = "twitter.png",
    color = "skyblue")
```
E voilà! Bem melhor, não é mesmo?
</div>

## Casos Interessantes
Algo muito peculiar que pode ser feito é tentar estabelecer correlações entre palavras, ver qual a frequência que elas são usadas uma atrás da outra ou na mesma frase com a função <span style = "font-family:Courier New">unnest_tokens(demoFreq)</span> e colocar <span style = "font-family:Courier New">token(demoFreq)</span> = "ngrams". 
Ao fazer isso, é possível colocar duas palavras e ver se elas aparecem juntas ocasionalmente e usar a função <span style = "font-family:Courier New">count(demoFreq)</span> .
OBS: Para "limpar" as análises de observações pouco interessantes ou de baixo impacto na pesquisa, basta utilizar a função <span style = "font-family:Courier New">separate(demoFreq)</span> e <span style = "font-family:Courier New">filter(demoFreq)</span> .

# Análise de Sentimentos
## O que é Análise de Sentimentos?
Trabalhar com análise de sentimentos é uma das aplicações mais importantes dentro da análise de texto. O cerne da questão é analisar qual é a conotação das palavras que aparecem no texto, utilizando das mesmas ferramentas computacionais mencionadas até o então, como as expressões regulares.

## Aplicações 
Utilizaremos esse tipo de análise para ver uma tendência à palavras, como se "Macron" é acompanhado de palavras positivas, se "França" possui alguma relação com "revoltas", "revoluções", e etc.
Contudo, suas aplicações não se restringem apenas para verificar a relação entre duas coisas, ela pode ser utilizada para ver o feedback de clientes em com um serviço ao analisar a conotação de cada review.

## Aplicações
Um ponto importante a se destacar é o algoritmo para realizar esse tipo de relação é que existem inúmeros algoritmos que fazem esse tipo de trabalho, então a eficiência das análises depende das funções utilizadas.
Como por exemplo, <span style = "font-family:Courier New">coreNLP</span>, <span style = "font-family:Courier New">cleanNLP</span> e <span style = "font-family:Courier New">sentimetr</span> .

# Trabalhando com "Clusters" 
Clusters, que são formas de organizar em grupos as análises feitas para os sentimentos. Você pode agrupar as análises de formas específicas, de acordo com o que você quer no momento, sendo não restringido apenas aos próprios sentimentos.

## Exemplo
Primeiramente, será necessário o pacote <span style = "font-family:Courier New">tidytext</span> para usar a função <span style = "font-family:Courier New">get_sentiment</span>. Com essa função, conseguimos ter alguns parâmetros para os grupos de sentimentos de raiva, felicidade, tristeza, etc... Por exemplo, existe o mais simples <span style = "font-family:Courier New">bing</span>, que apenas julga a conotação em verdadeiro ou falso para cada grupo, sem dar um "valor" para analisar o quão forte foi esse sentimento. Caso seja desejado uma escala para as análises, pode se utilizar <span style = "font-family:Courier New">affin</span>, que varia de -5 até 5 para a "força" do sentimento observado.

<span style = "font-family:Courier New">get_sentiment</span> ("nrc")
