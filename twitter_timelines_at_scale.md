### Twitter - Timeline at scale
https://www.infoq.com/presentations/Twitter-Timeline-Scalability/

http://highscalability.com/blog/2013/7/8/the-architecture-twitter-uses-to-deal-with-150m-active-users.html

http://highscalability.com/blog/2011/12/19/how-twitter-stores-250-million-tweets-a-day-using-mysql.html

Gist:
There are two types of timelines - user timeline(all tweets of particular user) and home timeline(temporal merge of users you are following). Timeline might have bussiness rules like visibility rules(no retweets, filter @ replies etc)

Timelines are also divided into two types - Targeted(Visiting twitter home page, home timeline API) and Queried(Search).
These timelines are also delivered in two ways Pull(visiting twitter home page, home timeline API, Search API), Push(Standing search query, mobile pushes, timeline stream)

 || Pull | Push|
 |---|---|---|
 |Targeted | twitter.com home timeline API | User / Site streams, Mobile Push|
 |Queried | Search API | Follow streams|

 300K QPS for timelines

Naive timeline materializations will be slow. Since the reads have to aggregate tweets from different users and read traffic is much much higher than write query, it makes sense to reduce computation during read path and do it on write path. Home timelines are pre-computed at write time. At read time, only bussiness rules like visibility filters etc are applied.

Write: When a new tweet is made, it goes to a fanout process - fanout checks "Social graph service" that stores the graph of all user relations(who follows who etc) to find the followers of user who made the tweet. The home timelines of all users are stored in redis cache - (key: userID, value: [tweetID]). The tweetID(not the tweet content) is appended to the end of the tweets list for all followers. Only 800 tweets are stored in cache. Lists of users who are active in last 30 days are only updated.

If the userId is not in redis cache, then once the list of followers is obtained from social graph servive, DB query has to be made to get the tweets from all persons the user is following and cache the user home timeline in redis.

Redis is sharded and every shard replicated to 3 replicas.

Read: During read, the timeline service get the home timeline from redis which is a list of tweet IDs. Then tweets are served by tweetyPie service and user objects are served by Gizmodo service. Both these services have their own caches.
Both the services support multi get to get all the tweets withs single api call. Faning out tweets from celebrities might be too slow and might cause race conditions when people start replying to the celbrity tweet and fanout of the reply is completed before the fanout of the original tweet. Solution is not to fan out celebrity tweets. Merge them during read path.

Tweet store:
Do not use time stamps as keys for sharding as it causes hot spots.
Uses snowflake for ID generation - https://github.com/twitter-archive/snowflake - Thrift service that uses Apache ZooKeeper to coordinate nodes and then generates 64-bit unique IDs
IDs can be sharded evenly. ID should also have time components and monotnically increasing if sorting by time is necessary. Snowflake can use time as first component.