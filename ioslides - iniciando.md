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
knitr::opts_chunk$set(echo = F)
```

```{r, echo = F, include = FALSE, eval = T, message = F}
library(pacman)

pacman::p_load(devtools, dplyr, rtweet, tm, RColorBrewer, cluster, fpc, httpuv,
              ggplot2, wordcloud, tidytext, SnowballC, htmlwidgets, lexiconPT,
              stringr, tidyverse, knitr, png, webshot, remotes, lubridate)

remotes::install_github("lchiffon/wordcloud2")
pacman::p_load(wordcloud2)
```

# {data-background=text_data_mining.jpg  data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}
<h2><span style="color: #000000;">Mineração: Primeiros passos</span><h2/>
## Mas afinal, por que minerar dados textuais?
<div class="columns-2">
<!-- ![dados_rede](dados_rede.jpg) -->
  <img src="dados_rede.jpg" height=460 width=450/ >
  <span style="font-size: small;">Fonte: Faculdade de Informática e Administração Paulista</span>
  
  
A mineração de texto é um recurso utilizado por cientistas de dados para coletar dados em forma de texto, visando obter informações relevantes.

A quantidade de dados em texto disponível é grande demais para ser ignorada, e ela cresce juntamente com o crescimento da internet. Então... por que ignorar tantos dados? Quanta informação podemos extrair deles?
</div>

## Método fácil | Pacotes
É comum que esse texto seja minerado de redes sociais ou de sites abertos a comentários, como sites de notícias, jornais, sites de streaming de vídeos como o YouTube, e alguns outros.

Devido ao fato de alguns sites serem minerados com frequência, existem alguns pacotes que podem facilitar ao minerar dados de alguns sites. Por exemplo, nós temos pacotes no R para minerar dados do Twitter, do Facebook e do Instagram.

```{r, eval = F, echo = T}
library(rtweet)
library(Rfacebook)
library(instaR)
```

## Extraindo dados do Twitter
O pacote <span style = "font-family:Courier New">rTweet</span> foi construído para extrair dados do Twitter, e uma função interessante dele é <span style = "font-family:Courier New">search_tweets</span>, que procura todos os tweets que seguem os padrões que definimos.
```{r, echo = T, eval=FALSE}
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


# {data-background=text_data_mining.jpg  data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}
<h2><span style="color: #000000;">Pré-processamento</span><h2/>

## Convertendo em corpus {.build}
```{r, eval = F, echo = T}
library(tm)
meuCorpus <- Corpus(VectorSource(meu_dataFrame)) # Criando um corpus a partir do dataframe
```

## Tokenização
*Tokenizar* um texto é fundamentalmente dividir ele em unidades menores que possam ser mais facilmente analisadas computacionalmente.
```{r, eval = T, echo = T, warning = F, message = F}
library(tokenizers)
```
Vejamos, por exemplo, como tokenizar um banco de dados de pangramas em português.
```{r, echo = T, eval = F}
View(pangramas)
```
```{r, eval = T, echo = F, warning = F}
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
```{r, eval = T, warning = F, message = F, echo = F}
options(max.print = 120)
```
```{r, eval = T, echo = T, warning = F, message = F}
tokenize_character_shingles(pangramas, n = 5, n_min = 5)[[1]]
```

## Tokenização | Funções {.build}
>Podemos também separar por palavras, e selecionar palavras a serem omitidas.
```{r, eval = T, echo = T, message = F, warning = F}
tokenize_words(pangramas, stopwords = stopwords("pt"))[[1]][c(1:5,7:9,11:16,17:22)]
```

>Ou até mesmo por frases:
```{r, eval = T, echo = T, message = F, warning = F}
tokenize_sentences(pangramas)[[1]][1:7]
```

## Limpando o banco de dados | Maiúsculas, minúsculas e pontuação {.build}
Para deixar o texto mais "legível" computacionalmente, é fundamental torná-lo mais homogêneo. O pacote <span style = "font-family:Courier New">tm</span> possui algumas funções interessantes para limpar dados em texto..
```{r, eval = F, echo = T}
meuCorpus <- tm_map(meuCorpus, tolower) # Tornando todas as letras minúsculas
meuCorpus <- tm_map(meuCorpus, removePunctuation) # Removendo a pontuação
meuCorpus <- tm_map(meuCorpus, removeWords, stopwords("pt")) # Removendo palavras
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

```{r, echo = F}
#dadosTwitter <- VCorpus(VectorSource(dadosTwitter$texto))
#dadosTwitter <- tm_map(dadosTwitter, removeWords, c(stopwords("pt"), "acho","aqui","bolsonaro","cê","dar","dia","entao","entrar","faz","fazer","fica","ficar","gente","indo","mim","nada","nao","nessa","pois","porque","pra","pro","quer","queria","quero","quis","sair","sao","sei","ser","sim","tá","tava","ter","tô","toda","tudo","vai","vcs","vem","ver","voce","vou"))
```

Após a limpeza dos dados, podemos avaliar o que os dados têm a dizer. Vamos lá!

# {data-background=text_data_mining.jpg  data-background-size=cover #azul .flexbox  .vcenter .centrobaixo}
<h2><span style="color: #000000;">Explorando os dados...</span><h2/>

## Wordclouds
Uma prática útil (e divertida!) é criar *wordclouds* (nuvens de palavras), que fornecem uma boa visualização dos termos que mais frequentes. Os pacotes <span style = "font-family:Courier New">wordcloud</span> e <span style = "font-family:Courier New">wordcloud2</span> são apropriados pra isso. Esses gráficos ordenam as palavras pela frequência com que aparecem nos dados.
```{r, eval = F, echo = T}
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
```{r fig.align="left", eval = T, warning = F, echo = T}
wordcloud(words = demoFreq$word, freq = demoFreq$freq, ordered.colors = TRUE)
```

## Wordclouds | Comparando os pacotes
```{r, fig.align="center", eval = T, echo = T}
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
```{r, echo = F, eval = F}
wordcloud2(demoFreq,
    figPath = system.file("examples/t.png", package = "wordcloud2"),
    color = "skyblue")
```

```{r, echo = T, eval = F}
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
```{r, echo = F, eval = T, warning = F, message = F}
#Criando o grafico
ggplot(rappi_plot, aes(x = reorder(word, -freq), y = freq)) +
  geom_bar(stat = "identity") + 
  theme(axis.text.x=element_text(angle=45, hjust=1)) +
  ggtitle("Grafico de barras com os termos mais frequentes") +
  labs(y="Frequencia", x = "Termos")

```

##Nuvem de palavras das 3 empresas
Lembrando que a função wordcloud 2 pode remover algumas palavras importantes para conseguir montar o desenho indicado no código, o que aconteceu na nuvem da empresa Uber eats que removeu as palavras 'desconto' e iFood.

##Gráfico temporal dos tweets que mencionam algumas das empresas
```{r, echo = F, eval = T, message = F, warning = F}
tweetsempresas <- read.csv("C:/Users/kevsd/Documents/UnB/Computação/Computação em Estatística/Análise e mineração de textos/tweetsempresas.csv")
tweetsempresas$created_at <- ymd_hms(tweetsempresas$created_at)
```
<div class="columns-2">
```{r, echo = T, eval = T, message = F, warning = F}
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

## Análise de Sentimentos | Exemplo (lexiconPT)
```{r echo=TRUE, eval = F}
library(lexiconPT)
ls('package:lexiconPT')
```
```{r, echo = F, eval = T}
ls('package:lexiconPT')
```

Dados do iFood e Uber Eats extraídos do Twitter com <span style = "font-family:Courier New">rtweet</span>:

```{r, echo = T, eval=FALSE}
iFood = search_tweets("iFood", n = 7500, include_rts = FALSE,lang = "pt")
iFood = iFood[,c('text')]
UberE = search_tweets("uber_eats", n = 7500, include_rts = FALSE,lang = "pt")
UberE = UberE[,c('text')]
Uber_and_ifood = rbind(UberE,iFood)
```

Vamos analisar qual dos tweets apresenta um padrão mais positivo ou negativo referente à empresa.

## Análise de Sentimentos | Limpando os tweets {.build}
```{r, echo = T, eval=FALSE}
f_clean_tweets <- function (tweets) {
  
  clean_tweets <- tweets$text
  clean_tweets = gsub('@\\w+', ' ', clean_tweets)# remove nomes pessoas
  clean_tweets = gsub('[[:punct:]]', ' ', clean_tweets)# remove pontuação
  clean_tweets = gsub('[[:digit:]]', ' ', clean_tweets)# remove números
  clean_tweets = gsub('[ \t]{2,}', ' ', clean_tweets)# remove espaços desnecessários
  clean_tweets = gsub('^\\s+|\\s+$', ' ', clean_tweets)##
  clean_tweets = gsub('<.*>', ' ', enc2native(clean_tweets))# removecaracteres especiais
  clean_tweets = gsub('\\n', ' ', clean_tweets)# remove quebra de linha
  clean_tweets = tolower(clean_tweets)# coloca tudo em minúsculo
  tweets$text <- clean_tweets
  tweets <- tweets[!duplicated(tweets$text),]# remove tweets duplicados
Uber_and_ifood = f_clean_tweets(Uber_and_ifood)
```
```{r, echo = F, eval = T, message = F, warning = F}
x = 'raiva app iFood'
x
```

## Análise de Sentimentos | Limpando os tweets {.build}
Retirar espaço em branco ' ' ocasionado pela função <span style = "font-family:Courier New">clean_tweets</span>:

```{r, echo = T, eval=FALSE}
for(i in 1:(length(Uber_and_ifood$text))){
  if(stringr::str_sub(Uber_and_ifood$text[i],start = 0,end = 1) == ' '){
    Uber_and_ifood$text[i] = str_sub(Uber_and_ifood$text[i],start = 2)
  }
}
```

## Análise de Sentimentos | Base de dados
```{r echo = T, eval=FALSE}
ifood_clean = Uber_and_ifood[cont:length(Uber_and_ifood$text),]
UberE_clean = Uber_and_ifood[1:(cont-1),]
```

Agora que temos os dados do Uber Eats e iFood limpos vamos à analise de sentimentos. 

```{r echo=TRUE}
# Carregando o data.base das palavras e respectivos sentimentos, indicados por polaridades (1,0,-1)
data("sentiLex_lem_PT02")
data_base_sentimentos = sentiLex_lem_PT02
data_base_sentimentos = data_base_sentimentos[,1:3]
data_base_sentimentos[1:9,]
```

## Análise de Sentimentos | Polaridade
A polaridade de cada palavra vai ser classificada da forma: 0 = neutra, 1 = positiva e -1 = negativa. Mas primeiro, precisamos atribuir a cada palavra dos tweets seus respectivos polos usando a função <span style = "font-family:Courier New">inner.join()</span>.
```{r echo=TRUE, eval=FALSE}
palavrasifood <- ifood_clean %>% unnest_tokens(ifood_clean, text)
palavras_UberE <- UberE_clean %>% unnest_tokens(UberE_clean, text)

#Alterar os nomes para, no próximo passo (inner.join), juntar por eles.

names(palavrasifood)[names(palavrasifood)== "ifood_clean"]<- "term"
names(palavras_UberE)[names(palavras_UberE)== "UberE_clean"]<- "term"
```
Desse modo nós temos todas as palavras de todos os tweets do iFood/UberE armazenados em um coluna, sendo cada palavra uma linha.

```{r, echo = F, eval = T, message = F, warning = F}
data.frame(term = c('raiva','app','iFood','paguei','loop'))
```

## Análise de sentimentos | inner.join()
Agora temos um <span style = "font-family:Courier New">dataframe</span> com cada palavra sendo uma linha. Podemos então executar o <span style = "font-family:Courier New">inner.join()</span> para análise de sentimentos.

```{r, echo = T, eval=FALSE}
sentimentos_ifood = palavras_ifood %>% 
    inner_join(dados_sentiment, by = "term")
sentimentos_UberE = palavras_UberE  %>% 
    inner_join(dados_sentiment, by = "term")
```
```{r, echo = F, eval = T, message = F, warning = F}
data.frame(term = c('raiva','bom','feio','perfeito','burrice'),
           grammar_category = c('N','Adj','Adj','Adj','N'),
           polarity = c(-1,1,-1,1,-1))
```

## Análise de sentimentos | inner.join()
Agora efetuando a média da polaridade em ambos iFood e UberEats com a média = 1 (perfeitamente positivo), média = 0 (neutro) e média = -1 (perfeitamente negativo)
```{r echo=TRUE, eval=FALSE}
media_ifood = mean(sentimentos_ifood$polarity)
media_UberE = mean(sentimentos_UberE$polarity)

resumo_sentimentos = data.frame(iFood = mean(sentimentos_ifood$polarity),
                                UberEats = mean(sentimentos_UberE$polarity),
                                row.names = 'Média Polaridade')
```

```{r, echo = F, eval = T, message = F, warning = F}
data.frame(iFood = c('-0.22533311'),
           UberEats = c('-0.31374237'),
           row.names = 'Média Polaridade')
```
Com essa demonstração é possivel observar que, com base no banco de dados de sentimentos <span style = "font-family:Courier New">lexiconPT</span>, ambos Uber Eats e iFood possuem uma média de tweets negativas com o Uber Eats sendo mais difamado do que o iFood.
