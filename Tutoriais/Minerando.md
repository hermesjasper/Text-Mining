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
