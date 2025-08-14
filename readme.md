# Compare Lists

A script to compare objects in two Lists

# Version 

1.0 Initial

# Global Script Setup
1. Create a Global Script called "CompareLists"
2. Add the input parameters below to the Global Script
   1. IdentifyerColumnName
   2. OriginalList
   3. UpdatedList
3. Add the output parameter below to the Global Script
   1. Changed
   2. Unchanged
4. Drag a *JavaScript* action into the script
5. Add the Javascript below unchanged into the JavaScript code property
```javascript
/* Stadium Script v1.0 https://github.com/stadium-software/utils-compare-lists */
let original = ~.Parameters.Input.OriginalList;
let updated = ~.Parameters.Input.UpdatedList;
let identifyer = ~.Parameters.Input.IdentifyerColumnName;
let changed = [];
let unchanged = [];
for (let i = 0; i < original.length; i++) {
    let originalItem = original[i];
    let updatedItem = findItem(updated, identifyer, originalItem[identifyer]);
    if (updatedItem) {
        if (!objectsAreSame(originalItem, updatedItem)) {
            changed.push(updatedItem);
        } else {
            unchanged.push(originalItem);
        }
    }
}
return {"changed": changed, "unchanged": unchanged};

/*functions*/
function findItem(arr, idField, id){
   return arr.find(item => item[idField] === id);
}
function objectsAreSame(x, y) {
   var objectsAreSame = true;
   for(var propertyName in x) {
      let xVal = x[propertyName];
      let yVal = y[propertyName];
      if (xVal !== yVal) {
         objectsAreSame = false;
         break;
      }
   }
   return objectsAreSame;
}
```
6. Drag a *SetValue* action into the Global Script and place it under the *JavaScript* action
   1. Target: = ~.Parameters.Output.Changed
   2. Value: = ~.Javascript.changed
7. Drag a *SetValue* action into the Global Script and place it under the *JavaScript* action
   1. Target: = ~.Parameters.Output.Unchanged
   2. Value: = ~.Javascript.unchanged

## Usage
1. Drag the script called "CompareLists" into a script or event handler
2. Enter values for the script input parameters
   1. IdentifyerColumnName: The name of an object that uniquely identifies an item in the two lists (e.g. ID)
   2. OriginalList: The initial list
   3. UpdatedList: The updated list
3. Result: The script returns two lists
    1. ~.CompareLists.Changed: The list of items that have changes in any of the objects 
    2. ~.CompareLists.Unchanged: The list of items where all objects are the same as the original 

**Note:** 

Data that was formatted for display purposes may be returned in a different format from the original, even though no changes were made to the actual data. 

This can unwittingly occurr when, for example, a Date is passed to a DatePicker Control as the control. In this case, an original format might need to be restored before using this script (see [How-To: Working With Dates](https://github.com/stadium-software/howto-date-formatting)). 

Input: Initial List
```json
[
    {
        "ID": 1,
        "FirstName": "Martina",
        "LastName": "Vaughn",
        "NoOfChildren": 5,
        "NoOfPets": 1,
        "StartDate": "1970-08-05",
        "EndDate": "1969-12-31",
        "Healthy": false,
        "Happy": true,
        "Subscription": "Subscribed"
    },
    {
        "ID": 2,
        "FirstName": "Bruce",
        "LastName": "Hester",
        "NoOfChildren": 10,
        "NoOfPets": 1,
        "StartDate": "2022-12-19", //note: the original date may not be in the same format as the updated date, even though the date is the same
        "EndDate": "2024-04-30", //note: the original date may not be in the same format as the updated date, even though the date is the same
        "Healthy": true,
        "Happy": false,
        "Subscription": "No data"
    }
]
```

Input: Changed List
```json
[
    {
        "ID": 1,
        "FirstName": "Martina1",
        "LastName": "Vaughn1",
        "NoOfChildren": "4",
        "NoOfPets": "2",
        "StartDate": "1970-08-06",
        "EndDate": "1969-12-08",
        "Healthy": true,
        "Happy": false,
        "Subscription": "Unsubscribed"
    },
    {
        "ID": 2,
        "FirstName": "Bruce",
        "LastName": "Hester",
        "NoOfChildren": 10,
        "NoOfPets": 1,
        "StartDate": "2022-12-19",
        "EndDate": "2024-04-30",
        "Healthy": true,
        "Happy": false,
        "Subscription": "No data"
    }
]
```

Output: Changed List
```json
[
    {
        "ID": 1,
        "FirstName": "Martina1",
        "LastName": "Vaughn1",
        "NoOfChildren": "4",
        "NoOfPets": "2",
        "StartDate": "1970-08-06",
        "EndDate": "1969-12-08",
        "Healthy": true,
        "Happy": false,
        "Subscription": "Unsubscribed"
    }
]
```

Output: Unchanged List
```json
[
    {
        "ID": 2,
        "FirstName": "Bruce",
        "LastName": "Hester",
        "NoOfChildren": 10,
        "NoOfPets": 1,
        "StartDate": "2022-12-19",
        "EndDate": "2024-04-30",
        "Healthy": true,
        "Happy": false,
        "Subscription": "No data"
    }
]
```