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

```
const newRow = {
  location: 'Istanbul, Turkey',
  year: 1976,
}
// Your code below!

db.run('INSERT INTO TemperatureData (location, year) VALUES ($location, $year)', {
  $location: newRow.location,
  $year: newRow.year
}, function(error) {
  // handle errors here!

  console.log(this.lastID);
});
```

* .each() method 
process every row returned from a database query.

```
db.each("SELECT * FROM Dog WHERE breed = 'Labrador'", 
  (error, row) => {
    // This gets called for every row our query returns
    console.log(`${row.name} is a good dog`);
  },
  (error, numberOfRows) => {
    // This gets called after each of our rows have been processed
    console.log(`There were ${numberOfRows} good dogs`);
});
```
```
const { printQueryResults, calculateAverages, addClimateRowToObject } = require('./utils');
const sqlite = require('sqlite3');

const db = new sqlite.Database('./db.sqlite');

const temperaturesByYear = {};

db.run('DROP TABLE IF EXISTS Average', error => {
  if (error) {
    throw error;
  }
  db.each('SELECT * FROM TemperatureData',
    (error, row) => {
      if (error) {
        throw error;
      }
      addClimateRowToObject(row, temperaturesByYear);
    }, 
    error => {
      if (error) {
        throw error;
      }
      const averageTemperatureByYear = calculateAverages(temperaturesByYear);
			printQueryResults(averageTemperatureByYear);
    }
  );
});
```

### Creating A New Table

```

const temperaturesByYear = {};

// start by wrapping all the code below in a serialize method

db.run('DROP TABLE IF EXISTS Average', error => {
  if (error) {
    throw error;
  }
  db.each('SELECT * FROM TemperatureData',
    (error, row) => {
      if (error) {
        throw error;
      }
      addClimateRowToObject(row, temperaturesByYear);
    }, 
    error => {
      if (error) {
        throw error;
      }
      const averageTemperatureByYear = calculateAverages(temperaturesByYear);
      db.run('CREATE TABLE Average (id INTEGER PRIMARY KEY, year INTEGER NOT NULL, temperature REAL NOT NULL)', logNodeError);
      averageTemperatureByYear.forEach(row => {
        db.run('INSERT INTO Average (year, temperature) VALUES ($year, $temp)', {
					$year: row.year,
        	$temp: row.temperature
      	}, err => {
          if (err) {
            console.log(err);
          }
        });
      });
    }
  );
});

```

### serial queries 
don’t want to try to INSERT data into a table that hasn’t been created yet. One way to avoid this issue is to write all of our code in nested callbacks,


```
db.serialize(() => {                                                                                                                          
  db.run("DROP TABLE Dog");
  db.run("CREATE TABLE Dog");
  db.run("INSERT INTO Dog (breed, name, owner, fur_color, fur_length) VALUES  ('Dachshund', 'Spike', 'Elizabeth', 'Brown', 'Short')");
});
```

```
const temperaturesByYear = {};

// start by wrapping all the code below in a serialize method
db.serialize(() => { 
  db.run('DROP TABLE IF EXISTS Average', error => {
    if (error) {
      throw error;
    }
  })
  db.run('CREATE TABLE Average (id INTEGER PRIMARY KEY, year INTEGER NOT NULL, temperature REAL NOT NULL)', logNodeError);
  db.each('SELECT * FROM TemperatureData',
    (error, row) => {
      if (error) {
        throw error;
      }
      addClimateRowToObject(row, temperaturesByYear);
    }, 
    error => {
      if (error) {
        throw error;
      }
      const averageTemperatureByYear = calculateAverages(temperaturesByYear);
      averageTemperatureByYear.forEach(row => {
        db.run('INSERT INTO Average (year, temperature) VALUES ($year, $temp)', {
          $year: row.year,
          $temp: row.temperature
        }, err => {
          if (err) {
            console.log(err);
          }
        });
      });
    db.all('SELECT * FROM Average',
    (error, row) => {
      printQueryResults(row)
    })
    });
  });
```


### error 

```

const newRow = {
  location: 'Istanbul, Turkey',
  year: 1976,
  tempAvg: 13.35
}

db.run('INSERT INTO TemperatureData (location, year, temp_avg) VALUES ($location, $year, $tempAvg)', {
  $location: newRow.location,
  $year: newRow.year,
  $tempAvg: newRow.tempAvg
}, function(error) {
  // handle errors here!
  if(error){
    return console.log(error);
  }
  
  console.log(this.lastID);
  
  db.get('SELECT * FROM TemperatureData WHERE id = $id', {
  		$id: this.lastID
	},
  function(error, row){
    printQueryResults(row);
 	});
});
```

# Modules 

### Export as
```
let specialty = '';
let isVegetarian = function() {
}; 
let isLowSodium = function() {
}; 
 
export { specialty as chefsSpecial, isVegetarian as isVeg, isLowSodium };
```

### Import as 

```
import { chefsSpecial, isVeg } from './menu';
```

```
import * as Carte from './menu';
 
Carte.chefsSpecial;
Carte.isVeg();
Carte.isLowSodium(); 
```

### Combine Export 

```
let specialty = '';
function isVegetarian() {
}; 
function isLowSodium() {
}; 
function isGlutenFree() {
};
 
export { specialty as chefsSpecial, isVegetarian as isVeg };
export default isGlutenFree;
```

### Combine Import 

```
import { specialty, isVegetarian, isLowSodium } from './menu';
 
import GlutenFree from './menu';
```