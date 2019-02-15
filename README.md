# Assignment3
The 3rd assignment for PB soft2019spring

# 1. Twitter data.
*For an executable version of these queries please see repo for last assignment https://github.com/cph-js284/MongoDbQueries*
Since I already build the actual program, I will just be handing in the queries and the response - I will be using the following format:
 - first part: the mongo shell query, this can be copied directly into a running MongoDb shell and executed (provided the names of the database and collection match)
 - second part: the C# version of the query.
 - third part: the response from MongoDb.
---------------------------------------------------------------------------------------------------------------
Question 1) How many Twitter users are in the database?

Mongo shell:  
db.TweetDocs.distinct("UserName").length

C#: 
var UserNameCounter = _collection.Distinct(x => x.UserName, _=> true).ToList().Count();

output: 
659775

----------------------------------------------------------------------------------------------------------------
Question 2) Which Twitter users link the most to other Twitter users? (Provide the top ten.)

Mongo shell:

```db.TweetDocs.aggregate([
{$match : {Text : {$regex: '@\\w+'}}},
{$project: {usr: "$UserName"}},
{$group: {_id: "$usr", NoL: {$sum:1}}},
{$sort: {"NoL": -1}},
{$limit:10}
])

C#:
var res = _collection.AsQueryable()
        .Where(x => x.Text.Contains("@"))
        .GroupBy(u => u.UserName)
        .Select(usr => new{UserName = usr.Key, TweetCount = usr.Count()})
        .OrderByDescending(c => c.TweetCount)
        .Take(10);

output:
User : lost_dog references other users 549 times
User : tweetpet references other users 310 times
User : VioletsCRUK references other users 251 times
User : what_bugs_u references other users 246 times
User : tsarnick references other users 245 times
User : SallytheShizzle references other users 229 times
User : mcraddictal references other users 217 times
User : Karen230683 references other users 216 times
User : keza34 references other users 211 times
User : TraceyHewins references other users 202 times
```
---------------------------------------------------------------------------------------------------------------------------------
