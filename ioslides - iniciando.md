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

pacman::p_load(devtools, dplyr, rtweet, tm, RColorBrewer, cluster, fpc, httpuv,
              ggplot2, wordcloud, tidytext, SnowballC,  htmlwidgets,
              stringr, tidyverse, knitr, png, webshot, remotes, lubridate)

remotes::install_github("lchiffon/wordcloud2")
pacman::p_load(wordcloud2)
```

# Mineração de texto: Dando os primeiros passos {data-background=text-mining-image.jpg  data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}
## Mas afinal, por que minerar dados textuais?
A mineração de texto é um recurso utilizado por cientistas de dados para coletar dados em forma de texto, visando obter informações relevantes.


## Método fácil | Pacotes
É comum que esse texto seja minerado de redes sociais ou de sites abertos a comentários, como sites de notícias, jornais, sites de streaming de vídeos como o YouTube, e alguns outros.

Devido ao fato de alguns sites serem minerados com frequência, existem alguns pacotes que podem facilitar ao minerar dados de alguns sites. Por exemplo, nós temos pacotes no R para minerar dados do Twitter, do Facebook e do Instagram.

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

Isso facilita muito o trabalho de mineração. Apesar disso, esses casos são as exceções, não a regra, e não podemos esperar que sempre teremos paocotes para nos ajudar.

## {data-background=emojipensativo.png data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}
<div style="text-align:right">
<h2>... e se não tivermos nenhum pacote?<h2/>
</div>

## Método "força-bruta" | Extraindo por HTML
Todas as páginas na rede possuem um código-fonte que pode ser acessado, e elas são sempre marcadas por HTML. A ideia de extrair dados em texto a partir do código-fonte é usar os padrões do HTML para extrair o código de interesse.

Para isso, devemos utilizar o R para entrar no código-fonte da página, e criar funções para que ele vá atrás desses padrões.


# Pré-processamento {data-background=text-mining-image.jpg  data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}

## Convertendo em corpus {.build}
```{r, eval = FALSE, echo = TRUE}
library(tm)
meuCorpus <- Corpus(VectorSource(meu_dataFrame)) # Criando um corpus a partir do dataframe
```

## Tokenização
*Tokenizar* um texto é fundamentalmente dividir ele em unidades menores que possam ser mais facilmente analisadas computacionalmente.
```{r, eval = TRUE, echo = TRUE, warning = FALSE, message = FALSE}
library(tokenizers)
```
Vejamos, por exemplo, como tokenizar um banco de dados de pangramas em português.
```{r, echo = TRUE, eval = FALSE}
View(pangramas)
```
```{r, eval = TRUE, echo = FALSE, warning = FALSE}
pangramas <- paste0("Jane quer LP, fax, CD, giz, TV e bom whisky.\n",
"TV faz quengo explodir com whisky JB.\n",
"Bancos fúteis pagavam-lhe queijo, whisky e xadrez.\n",
"Blitz prende ex-vesgo com cheque fajuto.\n",
"Um pequeno jabuti xereta viu dez cegonhas felizes.\n",
"Gazeta publica hoje no jornal uma breve nota de faxina na quermesse.\n",
"Zebras caolhas de Java querem passar fax para moças gigantes de New York.\n",
"Juiz faz com que whisky de malte baixe logo preço de venda.\n",
"Pangramas à beça jazem no sótão da memória-dervixe do faquir helênico.\n",
"João: “Vá às favas, judas, zerê caquético!”; Noé: “Eu? —Vá você, pinguço, linguinha, xibimba!.\n",
"Luís argüia à Júlia que «brações, fé, chá, óxido, pôr, zângão» eram palavras do português.\n",
"A famosa Kelly comeu pão infetado com arroz que o Barriga jantou vendo o filme da Wehrmacht xexelenta.\n",
"À noite, vovô Kowalsky vê o ímã cair no pé do pinguim queixoso e vovó põe açúcar no chá de tâmaras do jabuti feliz.\n")
  
pangramas
```

## Tokenização | Funções 
Podemos separar todos os dados apenas pela quantidade de letra que queremos.
```{r, eval = TRUE, warning = FALSE, message = FALSE, echo = FALSE}
options(max.print = 120)
```
```{r, eval = TRUE, echo = TRUE, warning = FALSE, message = FALSE}
tokenize_character_shingles(pangramas, n = 5, n_min = 5)[[1]]
```

## Tokenização | Funções {.build}
>Podemos também separar por palavras, e selecionar palavras a serem omitidas.
```{r, eval = TRUE, echo = TRUE, message = FALSE, warning = FALSE}
tokenize_words(pangramas, stopwords = stopwords("pt"))[[1]][1:18]
```

>Ou até mesmo por frases:
```{r, eval = TRUE, echo = TRUE, message = FALSE, warning = FALSE}
tokenize_sentences(pangramas)[[1]][1:7]
```

## Limpando o banco de dados | Maiúsculas, minúsculas e pontuação {.build}
Para deixar o texto mais "legível" computacionalmente, é fundamental torná-lo mais homogêneo, para evitar, por exemplo, que o computador interprete a mesma palavra como algo diferente por capitalização.
```{r, eval = FALSE, echo = TRUE}
meuCorpus <- tm_map(meuCorpus, tolower) # Tornando todas as letras minúsculas
meuCorpus <- tm_map(meuCorpus, removePunctuation) # Removendo a pontuação
meuCorpus <- tm_map(meuCorpus, removeWords, stopwords("pt")) # Removendo as stopwords
meuCorpus <- tm_map(meuCorpus, removeNumbers) # Removendo os números
```

## Limpando o banco de dados | Stopwords {.build}
Stopwords são palavras que, apesar de predominarem no texto, não dizem muito sobre ele. Geralmente, é ideal que sejam removidas para prosseguir com a análise.
```{r, echo = F, eval = T, message = F, warning = F}
options(max.print = 63)
```

```{r, echo = T, eval = T, message = F, warning = F}
stopwords("pt")
```

```{r, echo = FALSE}
#dadosTwitter <- VCorpus(VectorSource(dadosTwitter$texto))
#dadosTwitter <- tm_map(dadosTwitter, removeWords, c(stopwords("pt"), "acho","aqui","bolsonaro","cê","dar","dia","entao","entrar","faz","fazer","fica","ficar","gente","indo","mim","nada","nao","nessa","pois","porque","pra","pro","quer","queria","quero","quis","sair","sao","sei","ser","sim","tá","tava","ter","tô","toda","tudo","vai","vcs","vem","ver","voce","vou"))
```

Após a limpeza dos dados, podemos avaliar o que os dados têm a dizer. Vamos lá!

# Análise: Explorando os dados... {data-background=text-mining-image.jpg  data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}
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

## Wordclouds | Comparando os pacotes {.build}
```{r fig.align="left", eval = TRUE, warning = FALSE, echo = TRUE}
wordcloud(words = demoFreq$word, freq = demoFreq$freq, ordered.colors = TRUE)
```

## Wordclouds | Comparando os pacotes
```{r, fig.align="center", eval = TRUE, echo = TRUE}
wordcloud2(demoFreq, backgroundColor= "transparent", size = 0.9)
```


## Seria uma pena se... {.build}

\n
<h2>... fôssemos exigentes...</h2>

\n
<div class="centered">
<h2>... e quiséssemos...</h2>
</div>

\n
<div style="text-align:right">
<h2>... MAIS!!</h2>
</div>

## Wordclouds | Dando forma aos gráficos
<div class="columns-2">
<!-- ![wordcloudtwitterexample](wordcloudtwitterexample.png) -->
  <img src="wordcloudtwitterexample.png" height=420 width=490/ >

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

## Análise Competitiva no Twitter | Indústria de Delivery
  <!-- ![delivery](delivery.png) -->
  <img src="delivery.png" height=350 width=970/ >
  
## Análise Competitiva no Twitter | Palavras mais frequentes
```{r, echo = F, eval = T, warning = F, message = F}
rappi_plot <- as.data.frame(
      matrix(
        data = c("gkz310126052", 14317, "frete", 11362, "cupom", 11340, "ganhe", 9552, "desconto", 8813, "150", 6388,"código", 5706,"grátis", 5179, "r150", 4516, "app", 4025),
        byrow = TRUE,
        ncol = 2
    )
)
names(rappi_plot) <- c("word","freq")
```

```{r, echo = T, eval = F, warning = F, message = F}
#Criando o grafico
ggplot(rappi_plot, aes(x = reorder(word, -freq), y = freq)) +
  geom_bar(stat = "identity") + 
  theme(axis.text.x=element_text(angle=45, hjust=1)) +
  ggtitle("Grafico de barras com os termos mais frequentes") +
  labs(y="Frequencia", x = "Termos")

```

## Análise Competitiva no Twitter | Palavras mais frequentes
Percebemos que essas palavras possuem um certo padrão e parecem estar incluidas nos mesmos tweets, ou então em variações do mesmo tweet.
```{r, echo = F, eval = TRUE, warning = F, message = F}
#Criando o grafico
ggplot(rappi_plot, aes(x = reorder(word, -freq), y = freq)) +
  geom_bar(stat = "identity") + 
  theme(axis.text.x=element_text(angle=45, hjust=1)) +
  ggtitle("Grafico de barras com os termos mais frequentes") +
  labs(y="Frequencia", x = "Termos")

```

##Nuvem de palavras das 3 empresas
Lembrando que a função wordcloud 2 pode remover algumas palavras importantes para conseguir montar o desenho indicado no código, o que aconteceu na nuvem da empresa Uber eats que removeu as palavras 'desconto' e ifood.

##Gráfico temporal dos tweets que mencionam algumas das empresas
```{r, echo = F, eval = TRUE, message = F, warning = F}
tweetsempresas <- read.csv("C:/Users/kevsd/Documents/UnB/Computação/Computação em Estatística/Análise e mineração de textos/tweetsempresas.csv")
tweetsempresas$created_at <- ymd_hms(tweetsempresas$created_at)
```
<div class="columns-2">
```{r, echo = TRUE, eval = TRUE, message = FALSE, warning = FALSE}
tweetsempresas %>%
  dplyr::filter(created_at > "2019-10-01") %>%
  dplyr::group_by(empresa) %>%
  ts_plot("hour", trim = 1L) +
  ggplot2::geom_line() +
  ggplot2::theme_minimal() +
  ggplot2::theme(
    legend.title = ggplot2::element_blank(),
    legend.position = "bottom",
    plot.title = ggplot2::element_text(face = "bold")) +
  ggplot2::labs(
    x = NULL, y = NULL,
    title = "Frequência de tweets que mencionam as empresas estudadas",
    subtitle = "Tweets recolhidos de 6/11/19 até 10/11/19",
    caption = "Fonte: Dados coletados do API REST do Twitter via rTweet"
  ) +
  ggplot2::scale_color_manual(values=c('red', 'blue', 'green'))
```
</div>
##Essas empresas também usam a mineração de texto!
Como observado nesse estudo a empresa Rappi utiliza robôs para divulgar seus serviços e seus descontos, em alguns casos esses robôs são programados para encontrar combinações entre palavras que possivelmente indicam fome ou então apego aos descontos da empresa.
## Casos Interessantes
Algo muito peculiar que pode ser feito é tentar estabelecer correlações entre palavras, ver qual a frequência que elas são usadas uma atrás da outra ou na mesma frase com a função <span style = "font-family:Courier New">unnest_tokens(demoFreq)</span> e colocar <span style = "font-family:Courier New">token(demoFreq)</span> = "ngrams". 
Ao fazer isso, é possível colocar duas palavras e ver se elas aparecem juntas ocasionalmente e usar a função <span style = "font-family:Courier New">count(demoFreq)</span> .
OBS: Para "limpar" as análises de observações pouco interessantes ou de baixo impacto na pesquisa, basta utilizar a função <span style = "font-family:Courier New">separate(demoFreq)</span> e <span style = "font-family:Courier New">filter(demoFreq)</span> .

## Análise de Sentimentos
Trabalhar com análise de sentimentos é uma das aplicações mais importantes dentro da análise de texto. O cerne da questão é analisar qual é a conotação das palavras que aparecem no texto, utilizando das mesmas ferramentas computacionais mencionadas até o então, como as expressões regulares.

  <!-- ![reacts](reacts.png) -->
  <img src="reacts.png" height=250 width=970/ >

## Análise de Sentimentos | Aplicações
Utilizaremos esse tipo de análise para ver uma tendência à palavras, como se "Macron" é acompanhado de palavras positivas, se "França" possui alguma relação com "revoltas", "revoluções", e etc.

Contudo, suas aplicações não se restringem apenas para verificar a relação entre duas coisas, ela pode ser utilizada para ver o feedback de clientes em com um serviço ao analisar a conotação de cada review.

## Análise de Sentimentos | Aplicações
Primeiramente, será necessário o pacote <span style = "font-family:Courier New">tidytext</span> para usar a função <span style = "font-family:Courier New">get_sentiment</span>. Com essa função, conseguimos ter alguns parâmetros para os grupos de sentimentos de raiva, felicidade, tristeza, etc... Por exemplo, existe o mais simples <span style = "font-family:Courier New">bing</span>, que apenas julga a conotação em verdadeiro ou falso para cada grupo, sem dar um "valor" para analisar o quão forte foi esse sentimento. Caso seja desejado uma escala para as análises, pode se utilizar <span style = "font-family:Courier New">affin</span>, que varia de -5 até 5 para a "força" do sentimento observado.

<span style = "font-family:Courier New">get_sentiment</span> ("nrc")

## Análise de Sentimentos | Aplicações
Um ponto importante a se destacar é o algoritmo para realizar esse tipo de relação é que existem inúmeros algoritmos que fazem esse tipo de trabalho, então a eficiência das análises depende das funções utilizadas.
Como por exemplo, <span style = "font-family:Courier New">coreNLP</span>, <span style = "font-family:Courier New">cleanNLP</span> e <span style = "font-family:Courier New">sentimetr</span> .
