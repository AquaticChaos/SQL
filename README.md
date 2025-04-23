# College Entity Relationship Diagram Model

This relational diagram model visualizes the different tables pertaining to a generic college campus, and how each table correlates to each other, such as through one-to-many and one-to-one relationships.
Some key relationships includue the following:

- Student enrollments comprised of many classes over multiple semesters.
- Buildings comprising of multiple rooms, with some of those rooms being classrooms, who may have multiple professors and courses assigned.

![ERD Diagram](https://github.com/user-attachments/assets/b6701648-a669-4e61-a80f-682349f3919f)

# SQL Code & Business Question Snippets

The following three business questions involve a seperate database created by the fictitious StayWell company that manages multiple housing properties under its name. StayWell uses this database to keep track of their residents, property, service requests, offices, landlords, and more. 

## Residential Listing
```
The company that manages all of the listed Staywell properties wants a comprehensive list of all possible combinations of
residents if they were to share the property they reside in with one other resident to see if it cuts down on costs.
List both residents’ IDs and full names; the property ID, full address, and owner’s number and full name.

SELECT R1.RESIDENT_ID, (R1.FIRST_NAME + R1.SURNAME) AS FULL_NAME, 
R2.RESIDENT_ID, (R2.FIRST_NAME + R2.SURNAME) AS FULL NAME, 
R2.PROPERTY_ID, P.ADDRESS, O.OWNER_NUM, (O.FIRST_NAME + ‘ ’ + O.LAST_NAME) AS FULL_NAME 
  FROM RESIDENTS R1 INNER JOIN RESIDENTS R2 ON R1.PROPERTY_ID = R2.PROPERTY_ID 
  INNER JOIN PROPERTY P ON R2.PROPERTY_ID = P.PROPERTY_ID 
  INNER JOIN OWNER O ON P.OWNER_NUM = O.OWNER_NUM 
    WHERE R1.RESIDENT_ID < R2.RESIDENT_ID; 
```

## Resolving Service Requests
```
StayWell has received a large number of service requests / complaints regarding several properties that are either partially
resolved or completely unresolved. They want you to list the property address; as well as the number of reports, total
estimated hours, and total spent hours for each property. Only show the top five results where the total spent hours is less
than or equal to half of the total estimated hours (in other words, total spent hours * 2 is less than or equal to total
estimated hours). Then sort the results so that the most complaints appear first, followed by the least amount of spent
hours, followed by the most amount of estimated hours. 

SELECT TOP(5) P.ADDRESS, COUNT(R.PROPERTY_ID) AS NUM_REPTS,  
SUM(R.EST_HOURS) AS TOTAL_ESTHRS, SUM(R.SPENT_HOURS) AS TOTAL_SPENTHRS 
  FROM SERVICE_REQUEST R RIGHT JOIN PROPERTY P ON R.PROPERTY_ID = P.PROPERTY_ID 
    GROUP BY P.ADDRESS 
      HAVING SUM(R.SPENT_HOURS) * 2 <= SUM(R.EST_HOURS) 
        ORDER BY COUNT(R.PROPERTY_ID) DESC, 
        SUM(R.SPENT_HOURS) ASC, 
        SUM(R.EST_HOURS) DESC; 
```

## Residential Service Request Notification
```
Following the previous query, StayWell wants to notify the residents associated with any property that currently has
reports about a temporary change in monthly rent and when to next expect a service (if any). For each resident and report,
use an outer join to list the resident’s ID and full name (using the same method as the first business question); the
property ID, address, and monthly rent cost; and the service request’s description, status, and next service date (if any).
Be sure to only include residents/properties where the request status is not null or currently open, and if the next service
date is within a week to a month (31 days) of today’s date: October 2nd, 2019. Sort the results by date, with the soonest first. 

SELECT R.RESIDENT_ID, (R.FIRST_NAME + R.SURNAME) AS RES_FULLNAME, 
P.PROPERTY_ID, P.ADDRESS, P.MONTHLY_RENT, S.DESCRIPTION, S.STATUS, S.NEXT_SERVICE_DATE 
  FROM RESIDENTS R LEFT JOIN PROPERTY P ON R.PROPERTY_ID = P.PROPERTY_ID 
  LEFT JOIN SERVICE_REQUEST S ON P.PROPERTY_ID = S.PROPERTY_ID 
    WHERE (S.STATUS IS NOT NULL OR S.STATUS NOT LIKE ‘%Open%’) 
    AND DATEDIFF(day, ‘2019-10-02’, S.NEXT_SERVICE_DATE) BETWEEN 7 AND 31 
      ORDER BY NEXT_SERVICE_DATE; 
```
