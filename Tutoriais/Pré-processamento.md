Oioi, Kev aqui de novo. Aqui eu vou dar umas dicas de como você pode deixar seu banco de dados organizado pra análise de texto. É importante que você já tenha feito todo o trabalho de mineração e extração dos dados antes disso, mas isso é óbvio, então vamos lá.

## Convertendo em corpus
Assim como existem os dataframes, existem tipos de banco de dados apropriados pra análise de texto, e esse banco de dados é o corpus.

Corpus são documentos nos quais nós podemos aplicar as técnicas de análise e processamento com maior facilidade.

Quando você extrai os dados, eles geralmente retornam um dataframe. Pra converter ele em corpus, basta usar o pacote ```tm```.
```{r, eval = F, echo = T}
library(tm)
meuCorpus <- Corpus(VectorSource(meu_dataFrame)) # Criando um corpus a partir do dataframe
```

Agora, com ele convertido, vpodemos partir pra limpar os dados e tokenizar.

## Tokenização
Um texto natural pode ser facilmente compreendido e analisado linguisticamente por humanos. Porém, computacionalmente, é pouco eficiente mantermos o texto como ele originalmente está.

O processo de tokenização serve pra quebrar o texto em unidades menores pra que possamos aplicar as técnicas de processamento de linguagem. Dependendo do que queremos, podemos separar o texto por termos, palavras, frases, e etc.

Há alguns pacotes pra isso, e vamos usar o tokenizers.
```{r, eval = T, echo = T, warning = F, message = F}
library(tokenizers)
```

Agora, como aplicação, vamos criar e manipular um banco de dados com pangramas em português (só a título de informação, um pangrama é uma frase que contém todas as letras do alfabeto).

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
```
Vamos visualizar como ele está:
```
View(pangramas)

[1] "Jane quer LP, fax, CD, giz, TV e bom whisky.\nTV faz quengo explodir com whisky JB.\nBancos fúteis pagavam-lhe queijo, whisky e xadrez.\nBlitz prende ex-vesgo com cheque fajuto.\nUm pequeno jabuti xereta viu dez cegonhas felizes.\nGazeta publica hoje no jornal uma breve nota de faxina na quermesse.\nZebras caolhas de Java querem passar fax para moças gigantes de New York.\nJuiz faz com que whisky de malte baixe logo preço de venda.\nPangramas à beça jazem no sótão da memória-dervixe do faquir helênico.\nJoão: “Vá às favas, judas, zerê caquético!”; Noé: “Eu? —Vá você, pinguço, linguinha, xibimba!.\nLuís argüia à Júlia que «brações, fé, chá, óxido, pôr, zângão» eram palavras do português.\nA famosa Kelly comeu pão infetado com arroz que o Barriga jantou vendo o filme da Wehrmacht xexelenta.\nÀ noite, vovô Kowalsky vê o ímã cair no pé do pinguim queixoso e vovó põe açúcar no chá de tâmaras do jabuti feliz.\n"
```
Meio desordenado... não?

Podemos separar todos os dados apenas pela quantidade de letras que queremos. Vamos tentar, por exemplo, separar de 5 em 5 letras. Vejamos como fica:
```{r, eval = T, warning = F, message = F, echo = F}
tokenize_character_shingles(pangramas, n = 5, n_min = 5)[[1]]

  [1] "janeq" "anequ" "neque" "equer" "querl" "uerlp" "erlpf" "rlpfa" "lpfax"
 [10] "pfaxc" "faxcd" "axcdg" "xcdgi" "cdgiz" "dgizt" "giztv" "iztve" "ztveb"
 [19] "tvebo" "vebom" "ebomw" "bomwh" "omwhi" "mwhis" "whisk" "hisky" "iskyt"
 [28] "skytv" "kytvf" "ytvfa" "tvfaz" "vfazq" "fazqu" "azque" "zquen" "queng"
 [37] "uengo" "engoe" "ngoex" "goexp" "oexpl" "explo" "xplod" "plodi" "lodir"
 [46] "odirc" "dirco" "ircom" "rcomw" "comwh" "omwhi" "mwhis" "whisk" "hisky"
 [55] "iskyj" "skyjb" "kyjbb" "yjbba" "jbban" "bbanc" "banco" "ancos" "ncosf"
 [64] "cosfú" "osfút" "sfúte" "fútei" "úteis" "teisp" "eispa" "ispag" "spaga"
 [73] "pagav" "agava" "gavam" "avaml" "vamlh" "amlhe" "mlheq" "lhequ" "heque"
 [82] "equei" "queij" "ueijo" "eijow" "ijowh" "jowhi" "owhis" "whisk" "hisky"
 [91] "iskye" "skyex" "kyexa" "yexad" "exadr" "xadre" "adrez" "drezb" "rezbl"
[100] "ezbli" "zblit" "blitz" "litzp" "itzpr" "tzpre" "zpren" "prend" "rende"
[109] "endee" "ndeex" "deexv" "eexve" "exves" "xvesg" "vesgo" "esgoc" "sgoco"
[118] "gocom" "ocomc" "comch"
 [ reached getOption("max.print") -- omitted 580 entries ]
```
Bom, daqui a gente consegue perceber que a função não considera o caractere espaço " " como uma letra. Ou seja, ele separa literalmente por letras.

Podemos também separar por palavras, e selecionar palavras a serem omitidas. Nesse caso, vamos fazer com que a função omita as ```stopwords```, das quais vamos falar um pouco mais adiante.
```{r, eval = T, echo = T, message = F, warning = F}


> tokenize_words(pangramas, stopwords = stopwords("pt"))[[1]]
  [1] "jane"      "quer"      "lp"        "fax"       "cd"        "giz"      
  [7] "tv"        "bom"       "whisky"    "tv"        "faz"       "quengo"   
 [13] "explodir"  "whisky"    "jb"        "bancos"    "fúteis"    "pagavam"  
 [19] "queijo"    "whisky"    "xadrez"    "blitz"     "prende"    "ex"       
 [25] "vesgo"     "cheque"    "fajuto"    "pequeno"   "jabuti"    "xereta"   
 [31] "viu"       "dez"       "cegonhas"  "felizes"   "gazeta"    "publica"  
 [37] "hoje"      "jornal"    "breve"     "nota"      "faxina"    "quermesse"
 [43] "zebras"    "caolhas"   "java"      "querem"    "passar"    "fax"      
 [49] "moças"     "gigantes"  "new"       "york"      "juiz"      "faz"      
 [55] "whisky"    "malte"     "baixe"     "logo"      "preço"     "venda"    
 [61] "pangramas" "beça"      "jazem"     "sótão"     "memória"   "dervixe"  
 [67] "faquir"    "helênico"  "joão"      "vá"        "favas"     "judas"    
 [73] "zerê"      "caquético" "noé"       "vá"        "pinguço"   "linguinha"
 [79] "xibimba"   "luís"      "argüia"    "júlia"     "brações"   "fé"       
 [85] "chá"       "óxido"     "pôr"       "zângão"    "palavras"  "português"
 [91] "famosa"    "kelly"     "comeu"     "pão"       "infetado"  "arroz"    
 [97] "barriga"   "jantou"    "vendo"     "filme"     "wehrmacht" "xexelenta"
[103] "noite"     "vovô"      "kowalsky"  "vê"        "ímã"       "cair"     
[109] "pé"        "pinguim"   "queixoso"  "vovó"      "põe"       "açúcar"   
[115] "chá"       "tâmaras"   "jabuti"    "feliz"  
```
Agora reparem uma coisa interessante: os índices funciona perfeitamente com o pacote tokenizers, e isso nos dá uma boa liberdade pra escolher as palavras. A gente pode fazer a mesma coisa que acabou de fazer, mas mostrando apenas as palavras de 1 a 5, 7 a 9, 11 a 16 e de 17 a 22, usando um vetor numérico ```c(1:5,7:9,11:16,17:22)```.
```
tokenize_words(pangramas, stopwords = stopwords("pt"))[[1]][c(1:5,7:9,11:16,17:22)]

 [1] "jane"     "quer"     "lp"       "fax"      "cd"       "tv"       "bom"     
 [8] "whisky"   "faz"      "quengo"   "explodir" "whisky"   "jb"       "bancos"  
[15] "fúteis"   "pagavam"  "queijo"   "whisky"   "xadrez"   "blitz" 
```

É interessante também visualizarmos as sentenças. O pacote identifica o final de uma sentença por meio de um ```\n``` (que representa um salto de linha, mais ou menos como a tecla ```Ènter``` quando você escreve um texto). Podemos usar isso pra visualizar as frases:
```{r, eval = T, echo = T, message = F, warning = F}
tokenize_sentences(pangramas)[[1]][1:7]

[1] "Jane quer LP, fax, CD, giz, TV e bom whisky."                             
[2] "TV faz quengo. explodir com whisky JB."                                   
[3] "Bancos fúteis pagavam-lhe queijo, whisky e xadrez."                       
[4] "Blitz prende ex-vesgo com cheque fajuto."                                 
[5] "Um pequeno jabuti xereta viu dez cegonhas felizes."                       
[6] "Gazeta publica hoje no jornal uma breve nota de faxina na quermesse."     
[7] "Zebras caolhas de Java querem passar fax para moças gigantes de New York."
```

Pois bem, há outras coisas que temos que fazer pra limpar os dados, então vamos seguir adiante.

## Letras maiúsculas, minúsculas e pontuação
Algumas partes do texto podem atrapalhar muito quando você está processando o texto. Por exemplo, se você tentar fazer uma nuvem de palavras com o texto puro, é bem provável que apareça muitas conjunções, artigos e pronomes que, nos impedem de extrair o que o texto quer dizer de fato.

Essas palavras são o que chamamos de ```stopwords```. Geralmente, é ideal que sejam removidas para prosseguir com a análise.

```{r, echo = T, eval = T, message = F, warning = F}
stopwords("pt")

 [1] "de"     "a"      "o"      "que"    "e"      "do"     "da"     "em"    
 [9] "um"     "para"   "com"    "não"    "uma"    "os"     "no"     "se"    
[17] "na"     "por"    "mais"   "as"     "dos"    "como"   "mas"    "ao"    
[25] "ele"    "das"    "à"      "seu"    "sua"    "ou"     "quando" "muito" 
[33] "nos"    "já"     "eu"     "também" "só"     "pelo"   "pela"   "até"   
[41] "isso"   "ela"    "entre"  "depois" "sem"    "mesmo"  "aos"    "seus"  
[49] "quem"   "nas"    "me"     "esse"   "eles"   "você"   "essa"   "num"   
[57] "nem"    "suas"   "meu"    "às"     "minha"  "numa"   "pelos" 
 [ reached getOption("max.print") -- omitted 140 entries ]
```

O pacote ```tm```, que carregamos no começo desse tutorial, possui algumas funções interessantes para limpar dados em texto.
```{r, eval = F, echo = T}
meuCorpus <- tm_map(meuCorpus, tolower) # Tornando todas as letras minúsculas
meuCorpus <- tm_map(meuCorpus, removePunctuation) # Removendo a pontuação
meuCorpus <- tm_map(meuCorpus, removeWords, stopwords("pt")) # Removendo palavras
meuCorpus <- tm_map(meuCorpus, removeNumbers) # Removendo os números
```
Fazendo isso, você evita problemas posteriores. 

Enfim, vocês podem explorar mais esses pacotes no site do CRAN, mas a melhor forma, é claro, é colocar em prática pra ver como funciona.

Obrigado, e até mais!
