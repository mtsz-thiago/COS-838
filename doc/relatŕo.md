# Relatório do Projeto: Clusterização Hierárquica de Sequências de Nucleotídeos usando Apache/Spark

## Resumo

Este trabalho foi proposto como projeto final da disciplina COB-838.  Neste projeto utilizou-se a MLlib, uma biblioteca especializada em Machine Learning que faz parte do ecossistema Spark, para executar um fluxo para a clusterização de sequẽncias de nucleotídeos a partir da perfis de k-mers. Um aplicativo que implementa o fluxo foi criado e pode ser utilizado por pesquisadores  e bioinformatas  O fluxo proposto foi testado em uma base de dados composta por arquivos FASTA

## Introdução

Este projeto foi proposto e executado como projeto de conclusão da disciplina COB-838 do Programa de Sistemas de Computação da COPPE da UFRJ e tem por objetivo implementar um fluxo de Big Data no Framework Spark.

Uma das áreas que demanda por processamento de Big Data é a Bioinformática e esta foi a área escolhida para a aplicação. Em particular, o processamento de arquivos contendo sequências de nucleotídeos para achar similaridades entre as mesmas..

## Metodologia e Fluxo Proposto

A obtenção de agrupamentos de sequências de nucleotídeos por similaridade têm diversas aplicações em biologia, síntese de drogas específicas, prevenção e controle de infecções, etc. Existem diversos programas que constroem esses agrupamentos utilizando algorítmos diversos. O método proposto neste trabalho é baseado no uso de aprendizagem não-supervisionada de máquina utilizando os perfis de k-mers das sequências como features

### Perfis de k-mers como features

A avaliação a similaridade de sequências de nucleotídeos pode ser feita em termos de quais k-mers compõem essas sequência e qual a frequência que o mesmos aparecem nas frequẽncias. k-mers são subsequências de tamanho *k* obtidas a partir da sequência original usando uma janela deslizante:

```python
def calculaKmers(sequencia: str, k: int):
    kmers_list = []
    n_kmers = len(sequencia) - k + 1
    for i in range(0,n_kmers):
        kmer = sequencia[i:i+k]
        kmers_list.append(kmer)
    return kmers_list
```

Onde o parâmetro *k* é um hiperparâmetro que corresponde ao tamanho dos k-mers que serão utilizados.

Cada sequência apresentará uma correlação única entre um o k-mer específico e a frequência absoluta em que este k-mer aparece na mesma, chamada de *perfil de k-mer*. Uma vez obtidos os perfis, torna-se possível utilizá-los como features para uma tarefa de clusterização e por consequẽncia a obtenção de agrupamentos de sequências similares, que deseja-se que sejam dissimilares as de outros agrupamentos.

Estudos demonstram que o tamanho *k* dss k-mers tem influência direta na acurácia de modelos que fazem a classificação taxonômica das sequências contra bancos de referência [[1]])(. Assim espera-se que o uso de k-mers mais longos gere clusters mais coesos, isto é, compostos de sequências mais similares entre si e que diferem de sequências de outros clusters. O uso de k-mers mais longos, no entanto, aumenta a necessidade por melhores estratégias para o gerenciamento de memória e representação de features já que o número de possíveis k-mers $n_{k-mers}$ é:

$$
n_{k-mers} = 4^k
$$

Uma vez que sequências de nucleotídeos são compostas por 4 tipos de aminoácidos: A, T, C ou G. Apesar do número de possíveis k-mers crescer exponencialmente com o aumento do parâmetro *k*, o número de k-mers gerado por uma sequência de tamanho *L*, $n_{k-mers in seq.}$, é:

$$
n_{k-mers in seq.} = L - k + 1
$$

Ou seja, os perfis de k-mers geram features esparsas que podem ser eficientemente representadas usando vetores esparsos como os implementados pela classe SparseVector da biblioteca MLlib.

### Clusterização e avaliação da coezão dos clusters

Existem métodos hierárquicos e não-hierárquicos de clusterização. Uma vez que as sequências de nucleotídeos refletem a diversidade e os processos oriundos da Seleção Natural e do processo de especiação, as técnicas hierárquicas foram preferidas pois, espera-se, que gerem clusters com alguma interpretabilidade biológica.

Do algoritmos disponíveis, escolheu-se o Bisecting k-means, uma variante hierárquica do algoritmos k-means. O algoritmo Bisecting k-means tem apenas um hiperparâmetro *N*, o número de clusters folha. Isso facilita o processo de tunning do modelo que passa a envolver a varredura deste único parâmetro. A simplicidade do algoritmo e a disponibilidade de uma implementação na biblioteca MLlib da Apache justificam a escolha do algoritmo para este trabalho.

#### Métrica de distância

Além do algoritmo de clusterização em si, o projetista deve selecionar a métrica de distância apropriada ao problema em questão. No fluxo proposto neste trabalho utilizamos a distância euclidiana, A escolha dessa métrica põe mais peso em grandes disparidades de features de diferentes sequências, uma vez que toda diferença deve ser elevada ao quadrado antes de ser somada para obter a métrica escalar.

#### Avaliação da coesão dos clusters

A avaliação da coesão dos clusters foi feita usando a métrica de Shilhoette:

$$
score_{shilhoete} = \sum{i=1}{M}(b-a)/max(b,a)
$$

onde a: é a distância média intracluster e b a distância da amostra em questão ao cluster mais próximo que não contém a própria amostra.

Os valores do score de shilhoete varia entre 1, quando os clusters são considerados muito coesos, e -1, quando os clusters gerados ainda guardam muita semelhança entre si. Valores próximos de 1 são preferíveis.

### Apache/Spark MLlib

O processo proposto neste trabalho é paralelizável, uma vez que o perfil de k-mers de cada uma das sequência pode ser cálculado de forma independente. Além disso a necessidade de gerenciar o uso de memória de forma eficiente e o volume de dados que se pode processar justificam a escolha do Spark e as bibliotecas que compõem seu ecossistema para a implementação do fluxo. Em particular, nas etapas de clusterização, foi utilizada a MLlib que tem classes para a codificação das features, cálculo de distância, score de coesão e algoritmo de clusterização além de classes que facilitam o tunning do processo.

Quanto a leitura dos arquivos de entrada, não encontrou-se uma implementação para ler arquivos no formato FASTA e portanto foi necessário fazer uma implementação específica para o projeto.

## Aparato experiental e resultados

Para o desenvolvimento do fluxo foram utilizadas imagens Docker para simular um cluster Spark. Este cluster não apresentará o ganho de desempenho de um cluster real uma vez que concorre com outros processos da máquina, no entanto, demonstra as vantagens do framework no gerenciamento de paralelismos e gerenciamento de memória.

O Cluster simulado é composto de 4 containers Docker, um Master e 3 worker, 

## Conclusões

O Fluxo proposto foi implementado e executado usando Apache/Spark usando o cluster simulado descrito na seção anterior. Demonstrando a aplicabilidade do fluxo e a pertinência do framework Spark para executá-lo. Sendo este o objetivo primário do projeto que foi concluído no tempo disponível.

Como produto do projeto, foi desenvolvido um app que pode ser invocado de linha de comando para a execução do fluxo e cálculo dos clusters. Este aplicativo poderá ser utilizado no futuro por bioinformatas que queiram analisar as similaridades entre sequências.

### Dados de entrada

O fluxo proposto foi executado contra uma base de dados formada por 17 arquivos FASTA num total de 354,6 MB. No total essa base é composta por 1864 sequências distintas.

nenhuma modificação do fluxo seria necessária para processar volumes maiores, muito embora ajustes no tamanho e número de partições do RDD de entrada poderia tornar o processo mais eficiente.

Apesar de existirem bibliotecas de bioinformática que utilizam Spark como engine de processamento, como a [Adam](https://github.com/bigdatagenomics/adam), o fluxo proposto foi implementand usando apenas as biblioteca cliente do Spark para Python mantidas pela comunidade. O parse de arquivos de entrada FASTA e FASTAQ foi feito utilizando fluxos de transformações de Dataframes. Essa etapa do fluxo exigiu o processamento de linhas levando em consideração a ordem em que aparecem nos arquivos de entrada. O código de parse desses arquivos está descrito pelas funções:

```python
# pase das linhas Header
fasta_null_ids_df = fasta_plain_df.withColumn("seqID_wNull", seq2kmer_udf("row"))

# preenchimento dos valores nulos pelo último valor não nulo na coluna de ID
fasta_n_filter_df = fasta_null_ids_df.withColumn(
    "seqID", F.last('seqID_wNull', ignorenulls=True)\
    .over(Window\
    .orderBy('idx')\
    .rowsBetween(Window.unboundedPreceding, Window.currentRow)))

# removendo as linhas Header
fasta_df = fasta_n_filter_df\
                .where(F.col("seqID_wNull").isNull())\
                .select("seqID","row")\
                .toDF("seqID","seq")

# redução por chave para concatenar sequência
fasta_per_seq_df = fasta_df.rdd\
            .map(lambda r: (r.seqID, r.seq[0]))\
            .reduceByKey(lambda x,y:x+y)\
            .map(lambda x: Row(seqID=x[1],seq=x[0]))\
            .toDF(["seqID", "seq"])
```

Conceitualmente, arquivos FASTA são compostos de pares chave-valor em que as chaves são os identificadores das sequẽncias e o valor a sequência de aminoácidos em si. Não obstante, o último bloco de transofornações executa uma concatenação de partes de sequẽncias (que estavam em linhas distintas no arquivo original).

A função de *parse* das linhas header utiliza o primeiro caracter não " " como separador do ID, primeira parte da linha, dos comentários que compõem a segunda parte. Essas linhas se diferenciam das que contém as partes das sequências pois começam com o caracter ">". 

```python
def parse_fasta_id_line(l):
    """
    Desejamos extrair os IDs das sequências da linhas que começarem pelo caracter ''>'. Pelo padrão
    FASTA, o ID é a primeira palavra e é um campo composto por ID.CONTIG
    
    Input>
        l: Uma linha de um arquivo FASTA
    Return:
        ID: da sequência ignorando o número de contigs, ou None caso não seja uma linha de ID
    """
    if l[0][0] == ">":
        heaer_splits = l[0][1:].split(" ")[0]
        seq_id_split = heaer_splits.split(".")
        return seq_id_split[0]
    else:
        return None

seq2kmer_udf = udf(parse_fasta_id_line, T.StringType())
```

Um detalhe de implementação é a utilização da função *udf* importada do pacote *pyspark.sql.functions* como wrapper da função em si para que possa ser utilizada na geração de uma coluna através da função *withColumn*. O programador é obrigado a declarar o tipo de retorno utilizando os tipos disponíveis no pacote *pyspark.sql.types* para que o *pyspark* seja capaz de converteros tipos *Scala* em tipos *Python*.

O processo para os arquivos do tipo FASTAQ guarda semelhanças, mas esses arquivos são compostos por conjuntos de: identificador, sequência, linha de comentário e linha de qualidade de leitura e portanto o conjunto de transoformações utilizadas é ligeiramente diferente.

Nos experimentos executados, resolveu-se utilizar apenas arquivos FASTA que são mais comuns

### Trabalhos futuros

Trabalhos futuros podem explorar a qualidade do fluxo proposto do ponto de vista biológico ao compará-lo com outros existentes que executem a mesma tarefa. Variantes do fluxo podem ser testadas pela modificação do algoritmo de clusterização, métrica de avaliação de distância e variação do tamanho dos k-mers. O teste da performance do fluxo exige o aumento da base de entrada e o ajuste da infraestrutura adequada.
