# Source

https://www.udemy.com/course/elasticsearch-complete-guide

# Mapping Recomendations

1. Dynamic mapping is convenient, but not a good idea in production - use explicit mappings;
2. Save disk space with more optimized mappings when storing many documents;
3. Set dynamic to "strict", not false -> avoid surprises and unexpected results; YOU ARE ALWAYS IN CONTROL OF YOUR DOCUMENTS FIELDS.
4. Don't always map text fields as both text and keywords - typically only one is needed; Each mapping requires disk space;
    * Do you need to perform full-text searches? use text type
    * Do you need to aggregations, sorting or filtering on EXACT values? use type keyword
5. Disable coercion -> this only allows you to do the wrong thing;
6. For whole numbers, the integer data type might be enough;
    * long can store larger numbers, but also use more disk space
7. For decimal numbers, float might be enough; Double requires more disk space;
8. Set doc_values (mapping parameter) to false if you don't need sorting, aggregations and scripting;
9. Set norms (mapping parameter) to false if you don't need relevant scoring;
10. Set index (mapping parameter) to false if you don't need to filter on values - common for time series data;
11. This rules are probably not worth for less than 1 million documents - rule of thumb;
12. Worst case scenario: you'll need to perform reindexing;

# Queries and Relevance Scores

1. The query request lands on the **COORDINATING NODE** of a cluster;
2. By default, every node in the cluster may act as a coordinating node and may receive HTTP requests;
3. The coordinating node broadcasts the query to other nodes (shards or replicas) and then receive their results, merge them together and send them back to the client;
4. By default, the first **ten** results are explicit in HITS;
5. Each result has a field _score that says how well the matching record fits the query; By default, the matching records are sorted by their relevance score;
6. Okapi BM25 -> relevance scoring algorithm currently used by Elasticsearch. Older was TF / IDF. 
         * TF (Term Frequency)  -> how many times the term appear in the field for a give document?
         * Inverse Document Frequency (IDF) -> How often the term appears within the index? (i.e across all documents). If the term appears in many documents, it has a lower            relevance score
         * Field length norm -> How long is the field? Short fiels has more weight than a long field.
         * BM25 handles better stop words.
7. In the filter context (Do documento match the query?) , no relevance scores are calculated. In the Query Context, it looks for how How Well do Documents Match?
8. Filter context are common in dates, status, etc.
9. Match Queries passam pelo mesmo processo de standard analyzer que os documentos passam no momento de integração aos data structures de Inverted Index; Por isso conseguimos   encontrar full-text matches com letra maiúscula;
10. Term Queries **não** passam pelo mesmo processo de standard analyzer que os documentos passam no momento de integração aos data structures de Inverted Index; Por isso não   conseguimos encontrar exact matchs com letra maiúscula;
11. **O que encontramos no object _source não é o que o Elasticsearch de fato está procurando. Ele procura o que está dentro das estruturas (BKD Trees ou Inverted Index) do Apache Lucene. Para o caso de text fields, o que está no data structure Inverted Index do Apache Lucene são os fields que passam para pelo processo de Analyzer onde por default sofrem o filtro de lowercase no processo de tokenização**
12. Match Queries use a default boolean operator of **or**.
13. In Elasticsearch, you **denormalize** your data whenever possible because it leads to better performance. Kind of the opposite that what you would do on a Relational Database. Performance > disk space here.
14. Queries em elementes do tipo "nested" precisam incluir o objeto "nested" na query. Settando _source=false e expondo os inner_hits, podemos obter somente os resultados da query de nested objects sem retornar todo o documento;
15. Routing é uma forma de Elasticsearch saber qual shard um documento com certo ID deve ser adicionado. É necessário adicionar ?routing no momento de adição de documentos com relação parent-child com join_field. "name" dentro de um objeto "join_field" seta qual tipo de documento temos e "parent" irá determinar quem é o pai em uma relação de parent-child. The default routing behaviour é usar o id do documento como valor de routing e alimentar isto para uma hashing function. Precisamos sempre adicionar um query parameter "routing" com matching ID do parent para que a inserção de documentos ocorra de forma successfull. **Parent and child documents devem ser armazenados no mesmo shard, por isso devemos passar o query parameter routing**.








