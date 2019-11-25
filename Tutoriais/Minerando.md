Olá de novo!

Quando buscamos informações sobre algum fenômeno ou tema, na estatística, é comum que procuremos bancos de dados que satisfaçam as nossas buscas. Porém, nem sempre nós vamos encontrar esses bancos de dados disponíveis, e em vários casos, minerar esses dados é a única alternativa possível pra adquirirmos esses dados.


Esses dados no geral, são extraídos a partir do HTML da página (podendo ser acessado pelo código-fonte). Alguns pacotes podem facilitar esse processo, dependendo do site de onde queremos extrair os esses dados. É o caso do Twitter, Facebook e Instagram, para os quais o R já possui pacotes com funções prórprias para esse trabalho.

## Pacotes
```
library(rtweet)
library(Rfacebook)
library(instaR)
```

Se quisermos minerar tweets onde aparece uma citação a um termo, como "flamengo", por exemplo, podemos fazer isso com um simples comando na função ```search_tweets()```. 

```
dadosTwitter <- search_tweets("flamengo", n = 50, type = "recent")
```
Esse pacote apresenta algumas limitações no API, onde o limite de tweets que você pode extrair a cada 15 min é 18.000. Qualquer número maior que esse será bloqueado pelo programa.

## Pelo código-fonte
Pra fazer uma extração por meio do código-fonte da página, podemos usar alguns pacotes pra manipular isso.
```
library(httr)
library(xml2)
library(rvest)
```
#entro com um arquivo csv chamado nomes ao qual contera o nome dos artistas que escolhi ter musicas.
nomes<-read.table(file.choose(),header = TRUE)
tabelao<-data.frame(cod=c(1:10000),link=c(1),musica=c(1))
dados3<-data.frame(cod=c(1:10000),texto=c(1))
b=0

for (linha in 1:10) {
  
  urll<-as.character(nomes[linha,1])
  
  db1<-readLines(urll, encoding = "UTF-8")
  db1<-db1[114]
  
  corte<-str_locate(db1,fixed("cifras.html")) #diminuindo area de pesquisa e evitando confusao
  corte1<-str_locate(db1,fixed("src="))
  db1<-substr(db1,corte[1],corte1[1]-300)
  
  prox<-data.frame(str_locate_all(db1,fixed("href=")))  #estrutura onde começa as letras
  prox1<-data.frame(str_locate_all(db1,fixed("/\"")))   #estrutura onde termina as letras
  
  # substr(db1,prox[1,2],prox1[1,1]) usei pra verificar se tava cortando certo
  
  a<-max(dim(prox1))    #quantos elementos terão de musicas por autor
  for (i in 1:a) {
    tabelao$link[b+i]<-substr(db1,prox[i,2]+2,prox1[i,1])
  }
  b<-(b+a) #b recebe a para ele continuar juntando embaixo
}

#tabelao <- tabelao %>%                          ## auxilio pq alguns cantores tava com problema
#  mutate(c=str_count(tabelao$link,"\\/"))
#tabelao<-filter(tabelao,c==3)
#tabelao<-filter(tabelao,musica!=1)
#write.csv(tabelao,"C:\\Users\\gabriel\\Desktop\\tabelao.csv")  
b<-max(dim(tabelao))

for (linhat in 1:b) {
  urll1<-paste0("https://www.letras.mus.br",tabelao$link[linhat])  #colando site + nome do link
  dados<-readLines(urll1, encoding = "UTF-8")
  #oi<-str_locate(dados,"<title>")
  #oi<-na.omit(oi)
  #oi1<-str_locate(dados," - ")
  #oi1<-na.omit(oi1)
  #tabelao$musica[linhat]<-substr(dados,oi[2]+1,oi1[1]-1)  #colocando nome da musica na tabela
  
  tm<-nchar(dados[87])
  if (tm<100) {
    dados1<-dados[89]
  }else {dados1<-dados[87]
  }
  
  #aq é pra só pegar a letra da musica
  sigla<-str_locate(dados1,fixed("<p>"))   #localizando onde começa a letra
  sigla<-na.omit(sigla)
  sigla1<-str_locate(dados,fixed("</p> </div>"))   #localizando onde termina a letra
  sigla1<-na.omit(sigla1)
  dados3[linhat,2]<-substr(dados1,sigla[2]+1,sigla1[1]-1)
  dados3$cod[linhat]=tabelao$cod[linhat]
}
