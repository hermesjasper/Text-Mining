Olá pessoal! Kev aqui.

Essa página é um tutorial sobre como usar os pacotes wordcloud e wordcloud2 no R. Pois bem, sejam bem vindos. Antes de tudo, obviamente, vamos baixar os pacotes:
```
install.packages(c("remotes", "wordcloud"))
library(remotes)
library(wordcloud)

remotes::install_github("lchiffon/wordcloud2")
library(wordcloud2)
```

Caso o pacote remotes não funcione, tente trocá-lo por devtools:
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

#Rodando os códigos: demoFreq
