# Question 1

## Database Selection, Relational vs NoSQL

While selecting a database for your use case, you’ll need to think about what your data looks like, how you’ll query your data, and the scalability you’ll need in the future. While a relational database is a viable option most times, it is unsuited for large datasets and big data analysis. NoSQL databases are the preferred choice for large or ever-changing data sets as they can be scaled horizontally and are schema-less.

NoSQL databases are a family of DB which are not relational in nature and are grouped into four major categories viz. Document store, key-value pairs, Graph based and wide-column stores. They are suitable for specific [use cases](http://highscalability.com/blog/2011/6/20/35-use-cases-for-choosing-your-next-nosql-database.html).

### When to use Relational database

1. You’re working with complex queries and reports and **structured data**.
   With SQL you can build one script that retrieves and presents your data.

2. You have a high transaction oriented applications such as **CRM application**, **accounting** and **banking** software, and **e-commerce** platforms.

3. You are working on **financial applications** where consistency is critical and need to ensure ACID compliance.

4. You don’t anticipate a lot of changes or growth.
   Relational database is suitable where the schema is fixed and we are not dealing with huge amount of data.

### When to use NoSQL database

1. You are constantly adding new features, functions, data types. It’s difficult to predict how the application will grow over time.
   NoSQL database are suitable for storage of data like **article content**, **social media posts**, **sensor data**, and other types of **unstructured data** that won’t fit neatly into a table.

2. You are working with a highly flexible schema design or no predefined schema.
   The data modelling process is iterative and adaptive. Changing the structure or schema will not impact development cycles or create any downtime for the application.

3. You are not concerned about data consistency and 100% data integrity is not your top goal.
   For example, with social media platforms like **Instagram**, **Twitter**, **Facebook**, it isn’t important if everyone sees your new post at the exact same time, which means data consistency is not a priority.

4. You have a lot of data, many different data types, and your data needs will only grow over time.
   NoSQL makes it easy to store all different types of data together and without having to invest time into defining what type of data you’re storing in advance. NoSQL databases are much better suited for big data and real time analytics as flexibility is an important requirement which is fulfilled by their dynamic schema.

## Use cases more suitable for NoSQL DB

### Personalisation

Offering personalized user experience in mobile and web apps is to improve conversions.
Superior personalized experience demands a large volume of data as personalization is based entirely on data.
A relational database requires an expensive infrastructure to handle and manage the huge volume of data required
for personalization, whereas a NoSQL database can scale on demand and handle large chunks of data seamlessly.

### Real time experience

Most of the modern applications require fast and real-time data access to and from the server.
Real-time data access is not easy with relational databases whereas NoSQL databases can offer a real-time experience with low latency.

### Data analytics

Traditionally, operational and analytical databases were kept as 2 separate environments.
Applications used operational databases whereas business intelligence tools used analytical databases for reporting and analysis.

### CMS

CMS have been in existence for many years now and have been totally dependent on structured databases that lacked
speed and flexibility. With a NoSQL database, a CMS can store and read any type of data and present it to the customer
at the time of interaction. Since there is no need to define the data model upfront, a NoSQL supported CMS can store and manage user generated content on the fly.

### Cloud computing

Though most of the cloud computing platforms support structured databases, there is a clear shift towards NoSQL databases.
For any business that wishes to implement digital transformation, cloud computing is the best choice.
The most preferred databases for cloud platforms are NoSQL databases and hence NoSQL databases is the go-to choice for modern cloud development.&nbsp;

### Zero downtime

There are many applications which cannot afford to have a downtime no matter how minor it might be.
Applications powered by structured databases often suffer a downtime at the time of upgradation or migration.
Unstructured databases can be upgraded without having any downtime thus enabling the business to have a zero compromise towards customer experience.&nbsp;

### Customer 360 degree view

Customers expect a consistent experience on all the channels, and the enterprise always wants to provide the highest level of
customer services to their customers. However, as the number of products and services, channels and business units grows,
the fixed data model of relational databases forces enterprises to fragment customer data because different applications work with different customer data.
NoSQL document databases have a flexible data model that enables many applications to access the same customer data as well as add new attributes without affecting other applications.

# Question 2

# Possible Solution

### Design decision#1

- We shall store individual instances of ad-hoc events in a table.
- We shall not store the individual instances of a recurring event as rows in db, rather we will store a [RRULE](http://www.ietf.org/rfc/rfc2445.txt) string representing the recurrence pattern.
- The reason being that for recurring events with no end date it is not feasible to store all possible event instances. Also, updating recurring events will require more DB operations.
- We shall generate past and future event instances programmatically from RRULE string using third party [libraries](https://github.com/search?utf8=%E2%9C%93&q=rrule).

### Design details

- All scheduled events, irrespective of their ad-hoc or recurring nature, shall be stored in the Event table.
- Recurring events shall have a RRULE string defined for them.
- For ad-hoc event RRULE_String shall be NULL.
- We will convert all user-local date/time values to UTC for storage and processing, and only convert back to user-local time at the point of displaying back to the user.
- Always store the Start_Date and Duration (duration of event instance).
- The Start_Date/End_Date combination should always represent the entire possible range of dates matching the recurrence range.
- For recurrence patterns that include an End_Date explicitly (via UNTIL), store it as the End_Date of the range.
- For recurrence patterns that specify the number of instances (via COUNT), calculate the End_Date of the range and store that.
- For recurrence patterns with no End_Date or count, store the End_Date as a distant future date, such as the database's "max date" property (e.g., '9999-12-31')
- Always store a non-null End_Date, even for events with no Duration (store the same date/time for start and end in that case).

### Examples of RRULE string

- Weekly event scheduled for Tuesday until 2022-06-28.  
  **_FREQ=WEEKLY;INTERVAL=1;BYDAY=TU;UNTIL=20220628T000000Z_**

- Event scheduled for alternate days, It will repeat 5 times.  
  **FREQ=DAILY;INTERVAL=2;COUNT=5**

- Monthly event scheduled for first Monday. It will repeat forever.  
  **FREQ=MONTHLY;INTERVAL=1;BYDAY=1MO**

### Event Table columns

- EventID is the primary key in the table.
- The Start_Date and End_Date columns keep the start and end dates of end date for the recurrence pattern.
- In the case of regular(ad-hoc) events, these columns store actual start and end dates.
- Duration is typically stored as a time value in minutes.
- However, for recurring events they store the dates of the first and last occurrences of periodic events.
- The End_Date column as nullable, since users can configure recurring events with no end date. In this case, future occurrences up to a hypothetical end date (say for a year) would be shown in the UI.
- The Full_Day_Event column signifies if an event is a full-day event.
- Created_By and Created_Date columns store which user created an event and the date that event was created.

### Event Table

1. EventID PK int
2. Is_Recurring varchar
3. Event_Title varchar
4. Event_Description NULL varchar
5. Location NULL varchar
6. Start_Date datetime
7. End_Date NULL datetime
8. Full_Day_Event varchar
9. Duration int
10. Created_By varchar
11. Created_Date datetime
12. RRULE_String NULL varchar

### Event_Instance_Exception Table

1. ExceptionID PK int
2. Event_ID int FK >- Event.EventID
3. Exception_Date datetime

### Handling Exceptions of Recurring Events

- Exceptions are stored in the Event_Instance_Exception table.
- Exceptions are supported only as one-off instance instead of exception recurrence ranges and patterns.
- Exception_Date contains the date to be excluded from the recurring event.

### View/Query all events in their calendar at daily, weekly and monthly level

- Since we are storing the end date of last instance of event for recurring event, we can query the db to search for events whose end date is earlier than the end of duration (day, month and week). After retreiving such events we can perform following steps:
  - For each event, if Is_Recurring == true parse RRULE_String.
  - For each recurring event, generate new event instances (using the same Event model) that exist between the query start and end dates and match the recurrence pattern.
  - For each recurring event, if exception dates are supported, exclude those from the result set.
  - Return the final set of non-recurring events + recurring event instances

We can programmatically calculate the instances which lies within a given time frame. There are open-source libraries to work with RRULE strings. Below is a sample code using JS library **rrule.js**.

```javascript
import { RRule, RRuleSet, rrulestr } from "rrule";

// Create a rule:
const rule = new RRule({
  freq: RRule.WEEKLY,
  interval: 5,
  byweekday: [RRule.MO, RRule.FR],
  dtstart: new Date(Date.UTC(2012, 1, 1, 10, 30)),
  until: new Date(Date.UTC(2012, 12, 31)),
});

// Get all events scheduled for a particular day.
rule.between(new Date(Date.UTC(2021, 12, 27)), new Date(Date.UTC(2022, 1, 2)));

// Get all events in the week starting 27th Dec.
rule.between(new Date(Date.UTC(2021, 12, 27)), new Date(Date.UTC(2022, 1, 2)))[
  ("2012-08-27T10:30:00.000Z", "2012-08-31T10:30:00.000Z")
];

// Get all events scheduled for a particular day.
rule.between(new Date(Date.UTC(2021, 12, 27)), new Date(Date.UTC(2022, 1, 2)));
```

## Editing of recurring events

### Update or delete all instances of an event

Editing all instances means simply updating the Start_Date, Duration and/or RRULE_String of the existing event. Deleting means deleting the row in the database that represent the event.

### Update or delete only future event instances while keeping past instances

Split the original event into two separate events, each with its own distinct recurrence data. The original event is updated with a new end date and/or recurrence pattern, and the new event is a complete copy of the original (new row in the database) with a different event id and a different start date plus RRULE_String combination.

### Update or delete only a single selected event instance

To delete an instance you have to create an event exception (e.g. a new entry in the Event_Instance_Exception table). To update a single instance, we need an exception entry and also a new non-recurring event stored that contains the data unique to that instance.

# Question 3

- Download the executable file from [here](https://github.com/saurabhp75/python-executable/blob/main/dist/question3.exe), please ignore the warning and click "more info".
