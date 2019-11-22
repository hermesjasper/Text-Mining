Olá Pessoas!
  
  Luiz aqui, irei apresentar uma maneira de realizar análise de sentimentos por inner.join e utilizando a base de dados 'sentiLex_lem_PT02', contida no pacote 'lexiconPT'.

  A análise de sentimentos e constituída em 2 etapas, sendo a limpeza dos dados a 1° e consequentemente a atribuição de 'sentimentos' ao texto.
  
  Por cunho de aprendizado irei realizar um exemplo explicando cada passo desde à seleção do banco de dados à etapa final que é o sentimento extraido do texto. O exemplo terá como base a mineração dos últimos 10000 tweets do 'ifood' e avaliá-los em negativo ou positivos. Não será apresentado nenhum resultado, de tal modo que para vocês obterem-los possam modificar tanto a base dados dos sentimentos ou os próprios dados minerados.
  
Pacotes utilizados:
```
pacman::p_load(devtools, rtweet, tidytext,
               stringr, tidyverse, knitr, png, webshot, htmlwidgets,pkgbuild,lexiconPT, dplyr)
```
Seleção dos tweets, utilizando a função 'search_tweets'.
```
ifood = search_tweets("ifood", n = 10000, include_rts = FALSE,lang = "pt")
ifood = ifood[,c('text')]
```

Função para limpeza dos tweets(utilizando 'gsub').

```
f_clean_tweets <- function (tweets) {
  clean_tweets <- tweets$text
  clean_tweets = gsub('(RT|via)((?:\\b\\W*@\\w+)+)', ' ', clean_tweets)# remove retweet
  clean_tweets = gsub('@\\w+', ' ', clean_tweets)# remove nomes pessoas
  clean_tweets = gsub('[[:punct:]]', ' ', clean_tweets)# remove pontuaÃ§Ã£o
  clean_tweets = gsub('[[:digit:]]', ' ', clean_tweets)# remove nÃºmeros
  clean_tweets = gsub('http\\w+', ' ', clean_tweets)# remove html links
  clean_tweets = gsub('[ \t]{2,}', ' ', clean_tweets)# remove espaÃ§os desnecessÃ¡rios
  clean_tweets = gsub('^\\s+|\\s+$', ' ', clean_tweets)##
  clean_tweets = gsub('<.*>', ' ', enc2native(clean_tweets))# removecaracteres especiais
  clean_tweets = gsub('\\n', ' ', clean_tweets)# remove quebra de linha
  clean_tweets = gsub('[ \t]{2,}', ' ', clean_tweets)# remove espaÃ§os desnecessÃ¡rios
  clean_tweets = tolower(clean_tweets)# coloca tudo em minÃºsculo
  tweets$text <- clean_tweets
  tweets <- tweets[!duplicated(tweets$text),]# remove tweets duplicados
  tweets
}
ifood = f_clean_tweets(ifood)
```
Função criada para identificar os spams de promoção e oferta e substituí-los por 'NA'.
```
for(i in 1:(length(ifood$text))){
  if(stringr::str_sub(ifood$text[i],start = 0,end = 6) == 'cupons'){
    ifood$text[i] = '<>'
  }
  if(stringr::str_sub(ifood$text[i],start = 0,end = 5) == 'cupom'){
    ifood$text[i] = '<>'  
    
  }
  if(stringr::str_sub(ifood$text[i],start = 0,end = 6) == 'quer c'){
    ifood$text[i] = '<>'
  }
  if(ifood$text[i] == '<>'){
    ifood$text[i] = NA        
  }
}
```
Retirar as colunas que contém valores 'NA'.
```
ifood= ifood[complete.cases(ifood[,]),]
```
Retirar o primeiro espaço vazio ' ' que a função f_clean_tweets gerou (defeito da mesma).
```
for(i in 1:(length(ifood$text))){
  if(stringr::str_sub(ifood$text[i],start = 0,end = 1) == ' '){
    ifood$text[i] = str_sub(ifood$text[i],start = 2)
  }
}
```

Removendo stopwords.  
```
ifood$text = removeWords(ifood$text, stopwords(kind='pt'))
```
É de devida observação que a 'limpeza' dos textos é algo mutável, como o texto minerado é um tweet, tal limpeza pode não ser eficaz para outros textos. 
  
Agora que temos os dados do ifood limpos vamos à analise de sentimentos. Primeiro temos de carregar a base de dados das palavras e respectivos sentimentos que estão indicados por polaridade.
```
data("sentiLex_lem_PT02")
```
Base de dados que atribui sentimentos às palavras.

```
dados_sentiment = sentiLex_lem_PT02
```
Usar funçao do tidytext para criar uma linha para cada palavra de um tweet.

```
palavras_ifood <- ifood %>% unnest_tokens(ifood, text)
```

Alterar os nomes para, no próximo passo, juntar por eles.

```
names(palavras_ifood)[names(palavras_ifood)== "ifood"]<- "term"
```

Agora temos um data.frame com cada palavra sendo uma linha, assim podendo executar o inner.join para analise dos sentimentos.
```
sentimentos_ifood = palavras_ifood %>% 
  inner_join(dados_sentiment, by = "term")
```


Média da polaridade indicando o grau do sentimento, sendo mais proximo de 0 = neutro , +1 = perfeitamente posivivo e = -1 perfeitamente negativo.
```
media_ifood = mean(sentimentos_ifood$polarity)
```
Para vizualização ->
```
data.frame(ifood = c(media_ifood = mean(sentimentos_ifood$polarity))) 
``` 
