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

```
Mongo shell:  
db.TweetDocs.distinct("UserName").length

C#: 
var UserNameCounter = _collection.Distinct(x => x.UserName, _=> true).ToList().Count();

output: 
659775
```
----------------------------------------------------------------------------------------------------------------
Question 2) Which Twitter users link the most to other Twitter users? (Provide the top ten.)
```
Mongo shell:

db.TweetDocs.aggregate([
{$match : {Text : {$regex: '@\\w+'}}},
{$project: {usr: "$UserName"}},
{$group: {_id: "$usr", NoL: {$sum:1}}},
{$sort: {"NoL": -1}},
{$limit:10}
])

C#:
var res = _collection.AsQueryable()
        .Where(x => x.Text.Contains("@")) // <- there is an error here, I left it in to match the original code
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
Question 3) Who are the most mentioned Twitter users? (Provide the top five.)
```
Mongo shell:
db.TweetDocs.aggregate([
{$match: {Text: {$regex : '@\\w+'}}},
{$project: {TextArr: {$split : ["$Text", " "]}}},
{$project: {Name: {$arrayElemAt: ["$TextArr", 0]}}},
{$group: {_id:"$Name", NoRefs:{$sum:1}}},
{$sort: {"NoRefs": -1}},
{$limit: 5}
])

C#:
var res = _collection.AsQueryable().Where(x=>x.Text.Contains("@")).ToList();
Regex regex = new Regex(@"@[\w]+");
foreach (var item in res){
    var gr = regex.Match(item.Text).Groups;
        if(gr[0].Length>0){
            var tmp = gr[0].Value;
            if(store.TryGetValue(tmp, out int Counter)){
                store[tmp] = Counter + 1;
            }else{store[tmp] = 1;}
        }
}
var orderBy = store.OrderByDescending(x=>x.Value).Take(5);

output:
@mileycyrus - 4310
@tommcfly - 3767
@ddlovato - 3258
@DavidArchie - 1245
@Jonasbrothers - 1237
```
-------------------------------------------------------------------------------------------------------------------
Question 4) Who are the most active Twitter users (top ten)?
```
Mongo shell:
db.TweetDocs.aggregate([
{$group: {_id: "$UserName", NoTweets: {$sum:1}}},
{$sort: {"NoTweets": -1}},
{$limit: 10}
])

C#:
var res = _collection.AsQueryable()
        .GroupBy(u => u.UserName)
        .Select(usr => new{UserName = usr.Key, TweetCount = usr.Count()})
        .OrderByDescending(c => c.TweetCount)
        .Take(10);
       
output:
User : lost_dog has made 549 tweets
User : webwoke has made 345 tweets
User : tweetpet has made 310 tweets
User : SallytheShizzle has made 281 tweets
User : VioletsCRUK has made 279 tweets
User : mcraddictal has made 276 tweets
User : tsarnick has made 248 tweets
User : what_bugs_u has made 246 tweets
User : Karen230683 has made 238 tweets
User : DarkPiano has made 236 tweets
```
------------------------------------------------------------------------------------------------------------------------
Question 5) Who are the five most grumpy (most negative tweets) and the most happy (most positive tweets)?
```
Mongo shell (negative tweets):
db.TweetDocs.aggregate([
{$group: {_id:"$UserName", pol:{$sum:"$Polarity"}, NoTweets: {$sum:1}}},
{$sort: {"pol": -1}},
{$limit:5}
],{ allowDiskUse: true })

Mongo shell (positive tweets):
db.TweetDocs.aggregate([
{$group: {_id:"$UserName", pol:{$sum:"$Polarity"}, NoTweets: {$sum:1}}},
{$sort: {"pol": 1}},
{$limit:5}
],{ allowDiskUse: true })

C#:
var args = new AggregateOptions(){AllowDiskUse=true};
var Full = _collection.Aggregate(args)
        .Group(u=>u.UserName, grp=> new{
            UserName = grp.Key,
            PolScore = grp.Sum(y=>y.Polarity)
        }).ToList().OrderBy(x=>x.PolScore);
var Gres = Full.Take(5); //Grumpy
var Hres = Full.TakeLast(5); //Happy

output:
5 users with lowest polarity score (most grumpy)
iverissc - - 0
alanarubi - - 0
lizzyswims24 - - 0
lettherestdie - - 0
sarnwardhan - - 0

5 users with highest polarity score (least grumpy)
what_bugs_u - - 984
DarkPiano - - 924
VioletsCRUK - - 872
tsarnick - - 848
keza34 - - 844

```
-------------------------------------------------------------------------------------------------------------------------------
# 2. Modeling
