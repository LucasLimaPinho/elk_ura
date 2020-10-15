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
16. Documents should be in the same index - **Join Limitation**
17. Parents and child (and grandparents as so) should be in the same shard. Be careful with ?routing query parameter - **Join Limitation**
18. Must be only one join_field per index. A join_field can have as many relations as you want. New relations can be added after creating the index - **Join Limitation**
19. Documents can only have one parent, but multiple children is possible. E.g one employee can only have one department - **Join Limitation**
20. Each level of document relations adds an overhead to queries - **Join Field Performance Considerations**
21. More child documents makes has_child slower - **Join Field Performance Considerations**
22. More parent documents makes has_parent slower - **Join Field Performance Considerations**
23. Scenario where you should use join_field and relations: A one to many relationship between two document types where one type has many more documents than the other - **Join Field Performance Considerations**
24. Denormalize data instead of mapping document relationships. This is almost always the best approach. - **Join Field Performance Considerations**
25. Filters save computation time when we don't need to calculate relevance scores.
26. Fuzzy Queries are not analyzed.

# Aggregations

1. Metrics Aggregations - use Stats aggregations for min, max, avg, sum.
2. Bucket Aggregations - cria conjuntos para coletar os documentos. Terms é um tipo de aggregation utilizado para realizar Bucket Aggregation.
3. Utilizar parâmetro Missing para determinar o nome do bucket para documentos que não possuam o field utilizado para agregagação de terms.
4. value_count é a agregação utilizada para retornar o número de documentos. **Eles são uma aproximação devido performance issues**.
5. Counts are not always accurate devido a natureza distribuída do Elasticsearch com suas réplicas, shards e nodes. One index is distributed across multiple shards. Esta acurácia é muito modulada pelo parâmetro "size" da agregação "terms" -> É um tradeoff, quanto maior o size, maior sua acurácia, mas pior sua performance em relação a query. O valor default é 10.
6. doc_count_error_upper_bound -> esta key representa o máximo número de contagem de um termo que não foi parte do resultado final (uma métrica de quanto a contagem de documentos teve acurácia). Ele provê uma margem de erro. Na maioria dos cenários, a inacurácia do document count não será um problema mantendo o valor default de 10 para size como key do objeto "terms";
7. Podemos realizar uma restrição com key "query" antes de realizar a construção dos buckets.
8. Nested aggregations permite construir stats_aggregations (Metrics) para cada bucket criado pela BucketAggregation.
9. BucketAggregations may contain sub-aggregations. MetricsAggreation cannot.
10. Para date_range aggregations a regra é -> from dates are included, to dates are exluded -> (from, to]
11. É possível gerar as estatísticas diante do contexto gerado por cada bucket gerado pela agregação de date_range. 'keyed' = true e 'key' nos objetos de cada bucket permite nomear os buckets e cada um com suas stats_aggregations.
12. Histograms dynamically builds buckets from numeric fields based on a specified interval.







