# Power BI Guide
Noting down useful conventions and practices for Power BI.

Working document in progress.

### Naming conventions

1. Use human readable names. Avoid abbreviations.
2. Use spaces instead of underscores for naming tables, columns and measures. e.g. Sales 2020 instead of Sales_2020
3. Remove prefixes and suffixes from table names. e.g. DIM, FACT
4. Avoid camel case. e.g. Net Sales instead of NetSales

[//]: # 
#### DAX
1. Use [measure] and never use table[measure]
2. Use table[column] and never use [column]
3. Space before ‘(‘ and ‘)’ and any operand and operator
4. Space before an in-line argument
5. Definitions in the first row, including the assignment
6. Use ‘=’ to define calculated columns / tables
7. Use ‘:=’ to define measures

### Data Modeling

1. Do not duplicating dimension tables for different fact tables. This increases model size and reduces performance. Use 1 dimension table for multiple fact tables.

2. Always try to enable Query Folding.  
   Using a custom SQL query statement in Advanced Options breaks folding. Leave statement blank and use `Value.NativeQuery()` with `[EnableFolding=true]` enabled to enable query folding. Ensure that subsequent steps are foldable as well (right click and see if "View Native Query" is not greyed out.) https://wisedatadecisions.com/2021/07/20/query-folding-with-value-nativequery/
    ```

   
https://powerbi.microsoft.com/en-my/blog/best-practice-rules-to-improve-your-models-performance/