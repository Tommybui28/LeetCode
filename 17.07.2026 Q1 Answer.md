## Correct answer:
```
SELECT a.firstName, a.lastName, b.city, b.state 
FROM Person a 
LEFT JOIN Address b
    ON a.personId = b.personID
```
## My first answer.

```
SELECT a.firstName, a.lastName, b.city, b.state 
FROM 
Person AS a JOIN Address as B
WHERE a.personID = b.personID
```
## Why it was wrong.

For NULL return, requires LEFT Join.
Think of LEFT JOIN as saying "give me everyone on the left (Person), and attach their address if one exists — otherwise leave it null.

For INNER, LEFT, RIGHT, and FULL joins — yes, you need ON
