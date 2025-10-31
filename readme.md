# Compare Lists

A script that compares objects in two Lists. 

Two lists are passed into the script. When a common unique identifier is provided the objects of that item are compared to each other. When the SortLists parameter is set to true, the lists will be sorted before they ar compared. The script returns a boolean value indicating if the two lists are identical or not.

# Version 

1.1

## Changes
1.1 - Added option to sort lists before comparing them. Included support for deep lists (arrays within objects).

# Global Script Setup
1. Create a Global Script called "CompareLists"
2. Add the input parameters below to the Global Script
   1. IDColumn
   2. List1
   3. List2
   4. SortLists
3. Add the output parameter below to the Global Script
   1. Return
4. Drag a *JavaScript* action into the script
5. Add the Javascript below unchanged into the JavaScript code property
```javascript
/* Stadium Script v1.1 https://github.com/stadium-software/utils-compare-lists */
let set1 = ~.Parameters.Input.List1;
let set2 = ~.Parameters.Input.List2;
let idColumn = ~.Parameters.Input.IDColumn;
let sorted = ~.Parameters.Input.SortLists || false;
return compareArrays(set1, set2, idColumn, sorted);

function compareArrays(array1, array2, idProperty = null, sort = false) {
    if (array1 === array2) return true;
    if (!array1 || !array2) return false;

    const arr1 = typeof array1 === 'string' ? JSON.parse(array1) : array1;
    const arr2 = typeof array2 === 'string' ? JSON.parse(array2) : array2;

    if (!Array.isArray(arr1) || !Array.isArray(arr2)) return false;

    if (arr1.length !== arr2.length) return false;

    if (arr1.length === 0) return true;

    let copy1 = [...arr1];
    let copy2 = [...arr2];

    if (typeof copy1[0] === 'object' && copy1[0] !== null) {
        if (sort && idProperty) {
            copy1.sort((a, b) => compareByProperty(a, b, idProperty));
            copy2.sort((a, b) => compareByProperty(a, b, idProperty));

            for (let i = 0; i < copy1.length; i++) {
                if (!deepEqual(copy1[i], copy2[i])) {
                    return false;
                }
            }
        } else {
            for (const obj1 of copy1) {
                let foundMatch = false;
                for (const obj2 of copy2) {
                    if (deepEqual(obj1, obj2)) {
                        foundMatch = true;
                        break;
                    }
                }
                if (!foundMatch) {
                    return false;
                }
            }
        }
    } else {
        if (sort) {
            copy1.sort();
            copy2.sort();
        }

        for (let i = 0; i < copy1.length; i++) {
            if (!deepEqual(copy1[i], copy2[i])) {
                return false;
            }
        }
    }

    return true;
}

function getNestedProperty(obj, path) {
    return path.split('.').reduce((current, key) => {
        return current && current[key] !== undefined ? current[key] : undefined;
    }, obj);
}

function compareByProperty(a, b, property) {
    const valA = getNestedProperty(a, property);
    const valB = getNestedProperty(b, property);

    if (valA === valB) return 0;
    if (valA === null || valA === undefined) return 1;
    if (valB === null || valB === undefined) return -1;

    if (typeof valA === 'string' && typeof valB === 'string') {
        return valA.localeCompare(valB);
    }

    return valA < valB ? -1 : 1;
}

function deepEqual(a, b) {
    if (a === b) return true;

    if (a === null || b === null || a === undefined || b === undefined) {
        return a === b;
    }

    if (typeof a !== typeof b) return false;

    if (Array.isArray(a)) {
        if (!Array.isArray(b) || a.length !== b.length) return false;

        if (a.length === 0) return true;

        if (typeof a[0] === 'object' && a[0] !== null) {
            for (const objA of a) {
                let foundMatch = false;
                for (const objB of b) {
                    if (deepEqual(objA, objB)) {
                        foundMatch = true;
                        break;
                    }
                }
                if (!foundMatch) {
                    return false;
                }
            }
            return true;
        } else {
            for (let i = 0; i < a.length; i++) {
                if (!deepEqual(a[i], b[i])) return false;
            }
            return true;
        }
    }

    if (typeof a === 'object') {
        if (Array.isArray(b)) return false;

        const keysA = Object.keys(a);
        const keysB = Object.keys(b);

        if (keysA.length !== keysB.length) return false;

        for (const key of keysA) {
            if (!b.hasOwnProperty(key)) return false;
            if (!deepEqual(a[key], b[key])) return false;
        }
        return true;
    }

    return a === b;
}
```
6. Drag a *SetValue* action into the Global Script and place it under the *JavaScript* action
   1. Target: = ~.Parameters.Output.Return
   2. Value: = ~.Javascript

## Usage
1. Drag the script called "CompareLists" into a script or event handler
2. Enter values for the script input parameters
   1. IDColumn (optional): The name of an object that uniquely identifies an item in the two lists (e.g. ID)
   2. List1
   3. List2
   4. SortLists (optional, boolean, default false): Indicates if lists must be sorted before they are compared.
4. Result: The script returns a boolean value indicating if the two lists are identical or not.