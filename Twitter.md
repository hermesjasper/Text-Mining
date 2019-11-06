#Instalando os pacotes necessários
```{r}
require(devtools)
install_github("lchiffon/wordcloud2")

install.packages("pacman")
library(pacman)
pacman::p_load(rtweet, tm, RColorBrewer, cluster, fpc, httpuv, SnowballC, wordcloud, wordcloud2,
               tidytext, dplyr, stringr, rtweet, ggplot2)
```
# ####
# ####
#Carregando os Tweets
#Você vai precisar de uma conta no Twitter, e autorizar o uso dela para a mineração
#Limite de 18.000 tweets a cada 15 minutos
```{r}
twitterbolsonaro <- search_tweets("bolsonaro", n = 18000, include_rts = FALSE,lang = "pt")
```
#Rápida visualização - exemplo tirado da própria documentação da rtweet
```{r}
twitterbolsonaro %>%
  ts_plot("1 hours") +
  ggplot2::theme_minimal() +
  ggplot2::theme(plot.title = ggplot2::element_text(face = "bold")) +
  ggplot2::labs(
    x = NULL, y = NULL,
    title = "Frequência de tweets na hashtag #bitcoin",
    subtitle = "Tweets a cada 1 hora",
    caption = "\nFonte: Dados coletados da Twitter's REST API via rtweet"
  )
```
#####
#O trabalho de Mineração de Textos - Text Mining
##Criando e limpando o corpus
```{r}
corpusbolsonaro <- VCorpus(VectorSource(as.character(as.matrix(twitterbolsonaro$text))))
corpusbolsonaro <- tm_map(corpusbolsonaro, removePunctuation)
corpusbolsonaro <- tm_map(corpusbolsonaro, removeWords, c(stopwords("pt"), "acho","aqui","bolsonaro","cê","dar","dia","entao","entrar","faz","fazer","fica","ficar","gente","indo","mim","nada","nao","nessa","pois","porque","pra","pro","quer","queria","quero","quis","sair","sao","sei","ser","sim","tá","tava","ter","tô","toda","tudo","vai","vcs","vem","ver","voce","vou"))
```
#Primeira visualização
```{r}
wordcloud(corpusbolsonaro,min.freq=2,max.words=100, random.order=T, colors=brewer.pal(8,"Dark2"))
```
#Limpeza do texto com a D
##Limpando novamente o corpus
```{r}
bolsonaro_frequencia <- sort(colSums(as.matrix(removeSparseTerms(DocumentTermMatrix(corpusbolsonaro), 0.9995))), decreasing=TRUE) 
length(bolsonaro_frequencia) 
tail(bolsonaro_frequencia,10)
```
##Convertendo a matriz de frequencia em dataframe para o plot
```{r}
bolsonaro_plot <- data.frame(word=names(bolsonaro_frequencia), freq=bolsonaro_frequencia)  
bolsonaro_plot <- bolsonaro_plot[-c(1),]
```
#Criando o grafico
```{r}
ggplot(subset(bolsonaro_plot, bolsonaro_frequencia>1000), aes(x = reorder(word, -freq), y = freq)) +
  geom_bar(stat = "identity") + 
  theme(axis.text.x=element_text(angle=45, hjust=1)) +
  ggtitle("Grafico de barras com os termos mais frequentes") +
  labs(y="Frequencia", x = "Termos")
```
#Fazendo a nuvem de palavras
```{r}
wordcloud2(bolsonaro_plot) 
```
#####
#Aplicando um pouco de machine learning - Clustering
##Clustering 1 - Dendograma
```{r}
distancia <- dist(t(removeSparseTerms(removeSparseTerms(DocumentTermMatrix(corpusbolsonaro), 0.98), 0.95)), method="euclidian")
dendograma <- hclust(d=distancia, method="complete")
plot(dendograma, hang=-1,main = "Dendograma Tweets Bitcoin - Outspoken Market",
     xlab = "Distancia",
     ylab = "Altura")  
```
#Para ler melhor o Dendograma
```{r}
groups <- cutree(dendograma, k=3)
rect.hclust(dendograma, k=3, border="red")   
```
#Clustering 2 - K-Means
```{r}
kmeans_btc <- kmeans(distancia, 3)   
clusplot(as.matrix(distancia), kmeans_btc$cluster, color=T, shade=T, labels=3, lines=0,
         main = "K-Means Tweets Bitcoin - Outspoken Market",
         xlab = "PC1",
         ylab = "PC2")
```
