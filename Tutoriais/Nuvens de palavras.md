Olá pessoal! Kev aqui.

Essa página é um tutorial sobre como usar os pacotes ```wordcloud``` e ```wordcloud2``` no R. Pois bem, sejam bem vindos. Antes de tudo, obviamente, vamos baixar os pacotes:
```
install.packages(c("remotes", "wordcloud"))
library(remotes)
library(wordcloud)

remotes::install_github("lchiffon/wordcloud2")
library(wordcloud2)
```

Caso o pacote ```remotes``` não funcione, tente trocá-lo por ```devtools```:
```
install.packages(c("devtools", "wordcloud"))
library(devtoolss)
library(wordcloud)

devtools::install_github("lchiffon/wordcloud2")
library(wordcloud2)
```

Baixados e carregados os pacotes, vamos lá!

# Mas afinal... do que estamos falando?
Dentro da análise de dados textuais (nota: Eu não vou chamar análise de texto aqui, pois isso poderia ser confundido com algum termo linguístico relacionado a interpretação de texto ou seja lá o que for), nuvens de palavras (wordclouds) são como um carro-chefe, pois elas fornecem uma boa visualização do que está acontecendo no texto.

Isto é, as nuvens de palavras nos mostram com que frequência as palavras aparecem no texto por meio de um gráfico, onde, quanto maior for o número de vezes que a palavra aparece, maior a frequência que ela aparece no texto.

Entendeu? É simples! Mas vamos ver como funciona na prática.

# Sobre o banco de dados
Pois bem, antes de tudo, é importante fazer todo o processo de limpeza de dados que mostramos em outros tutoriais. Mas pra não mostrar isso aqui, eu vou usar o ```demoFreq```, que é um banco de dados que já está limpo e vem juntamente com o pacote ```wordcloud2```, e supor que já fizemos o processo de limpeza de dados necessário.

O demoFreq é um banco de dados com duas variáveis, words (palavras) e freq (frequência). E é assim que tem que estar o banco de dados para que você faça as nuvens de palavras. 
```
print(demoFreq)

                       word freq
oil                     oil   85
said                   said   73
prices               prices   48
opec                   opec   42
mln                     mln   31
the                     the   26
last                   last   24
bpd                     bpd   23
dlrs                   dlrs   23
crude                 crude   21
market               market   20
reuter               reuter   20
saudi                 saudi   18
will                   will   18
one                     one   17
barrel               barrel   15
kuwait               kuwait   14
new                     new   14
official           official   14
pct                     pct   14
price                 price   13
barrels             barrels   11
 [ reached 'max' / getOption("max.print") -- omitted 999 rows ]
 ```
 
# Usando as funções
O primeiro pacote, ```wordcloud```, tem os seguintes parâmetros a serem explorados:
```wordcloud(words,freq,scale=c(4,.5),min.freq=3,max.words=Inf,
	random.order=TRUE, random.color=FALSE, rot.per=.1,
	colors="black",ordered.colors=FALSE,use.r.layout=FALSE,
	fixed.asp=TRUE, ...)
 ```
 Os principais, entre todos eles, são ```words``` e ```freq```. Mas no geral, eles são:
 *word: 
