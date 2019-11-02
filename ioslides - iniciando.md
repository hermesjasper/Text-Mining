bol---
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
```{r}
require(devtools)
install_github("lchiffon/wordcloud2")

install.packages("pacman")
library(pacman)
pacman::p_load(rtweet, tm, RColorBrewer, cluster, fpc, httpuv, SnowballC,
              ggplot2, wordcloud, wordcloud2, tidytext, dplyr,
              stringr, tidyverse, knitr, png, webshot, htmlwidgets)
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
#dadosTwitter <- tm_map(dadosTwitter, removeWords, c(stopwords("pt"), "acho","aqui","cê","dar","dia","entao","entrar","faz","fazer","fica","ficar","gente","indo","mim","nada","nao","nessa","pois","porque","pra","pro","quer","queria","quero","quis","sair","sao","sei","ser","sim","tá","tava","ter","tô","toda","tudo","vai","vcs","vem","ver","voce","vou"))
```
```{r, echo = TRUE, eval = FALSE}
dadosTwitter <- VCorpus(VectorSource(dadosTwitter$texto)) #Convertendo em corpus
dadosTwitter <- tm_map(dadosTwitter, removeWords, stopwords("pt")) #Removendo stopwords
dadosTwitter <- tm_map(dadosTwitter, removePunctuation) #Removendo a pontuação
```
Após a limpeza dos dados, podemos avaliar o que os dados têm a dizer. Vamos lá!

## Word cloud: Um gráfico para textos

<div class="columns-2">
```{r, echo = FALSE}
#wordcloud2(dadosTwitter)
ggplot(cars, aes(speed, dist)) + geom_point()
```

Uma boa prática é criar *wordclouds* (nuvens de palavras), que fornecem uma boa visualização dos termos que mais frequentes. O pacote <span style = "font-family:Courier New">wordcloud2</span> foi construído para isso.
```{r, echo = TRUE, eval = FALSE}
wordcloud2(dadosTwitter)
#O único argumento necessário é o dataframe
#Basta que o banco esteja formatado
```
... Ok, mas será que não poderíamos deixar mais bonitinho? Hmmm, veremos.
</div>

## Word cloud: Um gráfico para textos
Podemos usar a função figPath para definir um formato que as palavras vão aparecer, usando uma imagem-base. Também podemos mudar alguns parâmetros da função.
```{r, echo = TRUE, eval = TRUE}
figPath <- system.file("twitter.png", package = "wordcloud2")
wordcloud2(demoFreq, figPath = "twitter.png", size = 1.5, color = "skyblue")
```
