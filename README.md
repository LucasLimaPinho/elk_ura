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








