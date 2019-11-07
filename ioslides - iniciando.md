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

```{r, echo = FALSE, include = FALSE, eval = TRUE}
options(repos="https://cran.rstudio.com")
install.packages("pacman", repos="https://cran.rstudio.com")
library(pacman)

pacman::p_load(devtools, rtweet, tm, RColorBrewer, cluster, fpc, httpuv, SnowballC,
              ggplot2, wordcloud, wordcloud2, tidytext,
              stringr, tidyverse, knitr, png, webshot, htmlwidgets)

devtools::install_github("lchiffon/wordcloud2")
```

## Mineração de texto
A mineração de texto é um recurso utilizado por cientistas de dados para coletar dados em forma de texto, visando obter informações relevantes.

Existem alguns pacotes no R com mecanismos pra coletar dados de sites específicos, como o Twitter, o Facebook e o Instagram. Apesar disso, na maioria das vezes não vão existir pacotes predefinidos para os dados de interesse, e os dados devem ser obtidos à partir do código-fonte da página (em HTML).

```{r, eval = FALSE, echo = TRUE}
library(rtweet)
library(Rfacebook)
library(instaR)
```

## Extraindo dados do Twitter
O pacote <span style = "font-family:Courier New">rTweet</span> foi construído para extrair dados do Twitter. Para pesquisar por algum termo, por exemplo, podemos usar a função <span style = "font-family:Courier New">search_tweets:</span>
```{r, echo = TRUE, eval=FALSE}
dadosTwitter <- search_tweets(q, n = 100, type = "recent", include_rts = TRUE,
  geocode = NULL, max_id = NULL, parse = TRUE, token = NULL,
  retryonratelimit = FALSE, verbose = TRUE, lang = "...")
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


## Nuvens de palavras: Explorando...

Podemos fazer um exemplo bem simples, rodando <span style = "font-family:Courier New">wordcloud(demoFreq\$word, demoFreq\$freq)</span> e <span style = "font-family:Courier New">wordcloud2(demoFreq)</span> no console, e então comparar os dois pacotes.
<span style = "position:center">
```{r, echo = FALSE, eval = TRUE}
wordcloud(words = demoFreq$word, freq = demoFreq$freq, colors = brewer.pal(8,"Dark2"))
wordcloud2(demoFreq, backgroundColor= "transparent", size = 1)
```
</span>

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

## Nuvens de palavras: Explorando...

Esses recursos são vastos e podem ser amplamente explorados, e como vimos, eles nos dizem muito sobre o que está acontecendo. Mas enfim, vamos prosseguir para o próximo tema.

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
Agora, veremos o que são os Clusters, que são formas de organizar em grupos as análises feitas para os sentimentos. Ou seja, você pode agrupar as análises de formas mais ou menos específicas, de acordo com o que você quer no momento, sendo não restringido apenas aos próprios sentimentos.

## Exemplo
Primeiramente, será necessário o pacote <span style = "font-family:Courier New">tidytext</span> para usar a função <span style = "font-family:Courier New">get_sentiment</span>. Com essa função, conseguimos ter alguns parâmetros para os grupos de sentimentos de raiva, felicidade, tristeza, etc... Por exemplo, existe o mais simples <span style = "font-family:Courier New">bing</span>, que apenas julga a conotação em verdadeiro ou falso para cada grupo, sem dar um "valor" para analisar o quão forte foi esse sentimento. Caso seja desejado uma escala para as análises, pode se utilizar <span style = "font-family:Courier New">affin</span>, que varia de -5 até 5 para a "força" do sentimento observado.

<span style = "font-family:Courier New">get_sentiment</span> ("nrc")
