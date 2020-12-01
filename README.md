# IMPORT & OPEN DB

```
const sqlite3 = require('sqlite3');
const db = new sqlite3.Database('./db.sqlite');

```
### Fetch data

* .find() method


```
db.all('SELECT * FROM TemperatureData ORDER BY year',(error,rows) => {
        if(error){
            throw error;
        }
        printQueryResults( rows.find(row=> row.id === 1) );
    }
)
```

* .get() method
return single row from database

```
db.get("SELECT * FROM Dog WHERE owner_name = 'Charlie'",(error,row) => {
    printQueryResults(row);
});
```

```
const ids = [1,25,45,100,360,382];

ids.forEach(id => {
        db.get(
            "SELECT * FROM TemperatureData WHERE id = $id",
            { $id: id } ,
            (err,rows) => { printQueryResults(rows); }
            )
        }
)
```

* .run() method
does not return a value
