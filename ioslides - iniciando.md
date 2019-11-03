---
title: 'Mineração e análise de textos no <img src="Rstudio_logo.png" height=90/>'
date: "8 de novembro de 2018"
output:
  ioslides_presentation:
    logo: UnB_logo.png
    widescreen: true
    css: Estilos.css
---

```{r, include = FALSE}
setwd(getwd())
knitr::opts_chunk$set(echo = FALSE)
```

## Pacotes
```{r, eval = TRUE}
options(repos="https://cran.rstudio.com")
install.packages("pacman", repos="https://cran.rstudio.com")
library(pacman)

pacman::p_load(devtools, rtweet, tm, RColorBrewer, cluster, fpc, httpuv, SnowballC,
              ggplot2, wordcloud, wordcloud2, tidytext,
              stringr, tidyverse, knitr, png, webshot, htmlwidgets)
devtools::install_github("lchiffon/wordcloud2")
```

## Afinal, do que se tratam a análise e a mineração de texto?

<div class="columns-2">
  <!-- ![big_data_wordcloud](big_data_wordcloud.png) -->
  <img src="big_data_wordcloud.png" height=450 width=450/ >

  - A mineração de textos se trata de extrair computacionalmente dados em texto;
  - A análise de textos, por sua vez, é transformar os dados em informação que possa ser usada para algum fim;
</div>

## Extraindo dados do Twitter
O pacote <span style = "font-family:Courier New">rTweet</span> foi construído pra extrair dados do Twitter. Para pesquisar por algum termo, por exemplo, podemos usar a função <span style = "font-family:Courier New">search_tweets:</span>
```{r, echo = TRUE, eval=FALSE}
dadosTwitter <- search_tweets(q, n = 100, type = "recent", include_rts = TRUE,
  geocode = NULL, max_id = NULL, parse = TRUE, token = NULL,
  retryonratelimit = FALSE, verbose = TRUE, lang = "...")
```
Assim, conseguimos dados com todos os *tweets* que citam algum termo que definimos. Por exemplo, se procurarmos por *insira aqui um termo*, obteremos:

```{r, echo = TRUE, eval = FALSE}
dadosTwitter <- search_tweets("insira aqui um termo",
                              include_rts = FALSE,
                              n = 18000,
                              lang = "pt")
```

## Extraindo dados do Twitter
```{r, echo = FALSE, eval = TRUE}
#dadosTwitter
```
## Limpando os dados
Antes de tudo, é **necessário** limpar o banco de dados. Para isso, temos que converter o banco de dados em Corpus e remover as *stopwords* e a pontuação.

```{r, echo = FALSE}
#dadosTwitter <- VCorpus(VectorSource(dadosTwitter$texto))
#dadosTwitter <- tm_map(dadosTwitter, removeWords, c(stopwords("pt"), "acho","aqui","bolsonaro","cê","dar","dia","entao","entrar","faz","fazer","fica","ficar","gente","indo","mim","nada","nao","nessa","pois","porque","pra","pro","quer","queria","quero","quis","sair","sao","sei","ser","sim","tá","tava","ter","tô","toda","tudo","vai","vcs","vem","ver","voce","vou"))
```
```{r, echo = TRUE, eval = FALSE}
dadosTwitter <- VCorpus(VectorSource(dadosTwitter$texto)) #Convertendo em corpus
dadosTwitter <- tm_map(dadosTwitter, removeWords, stopwords("pt")) #Removendo stopwords
dadosTwitter <- tm_map(dadosTwitter, removePunctuation) #Removendo a pontuação
```
Após a limpeza dos dados, podemos avaliar o que os dados têm a dizer. Vamos lá!

# Nuvens de palavras: Explorando...
## Nuvens de palavras: Explorando...
Uma prática útil (e divertida!) é criar *wordclouds* (nuvens de palavras), que fornecem uma boa visualização dos termos que mais frequentes. Pra isso é que servem os pacotes <span style = "font-family:Courier New">wordcloud</span> e <span style = "font-family:Courier New">wordcloud2</span>.
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


## Nuvens de palavras: Explorando...
Podemos fazer um exemplo bem simples, rodando <span style = "font-family:Courier New">wordcloud2(demoFreq)</span> no console:
```{r, echo = FALSE, eval = TRUE}
wordcloud2(demoFreq, ellipticity = 0)
```

## Nuvens de palavras: Explorando...
<div class="columns-2">
  <!-- ![wordcloudtwitterexample](wordcloudtwitterexample.png) -->
  <img src="wordcloudtwitterexample.png" height=400 width=450/ >
  
  
Ok, está legal, mas podemos melhorar um pouco, não?

Vamos tentar usar alguns parâmetros, para dar forma ao gráfico.
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
##  Nuvens de palavras: Explorando...
Esses recursos são vastos e podem ser amplamente explorados, e como vimos, eles nos dizem muito sobre o que está acontecendo. Mas enfim, vamos prosseguir para o próximo tema.
