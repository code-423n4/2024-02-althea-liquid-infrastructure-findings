### [L-01] `delete` keyword doesn't delete all elements in an array

The `delete` keyword on the array only sets the length of the array to 0, making the array element inaccessible. But, they are not deleted from the storage.
Consider iterating through the element overwrite to a new value. If the new value's length is less than the existing value, pop up those additional items from the array.


