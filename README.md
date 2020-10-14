https://www.udemy.com/course/elasticsearch-complete-guide


1. Dynamic mapping is conveniente, but not a good idea in production - use explicit mappings;
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






