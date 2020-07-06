# Reddit Streaming Data Lake

## Overview

AWS provides several key services for an easy way to quickly deploy and manage data streaming in the cloud.  Reddit is a popular social news aggregation, web content rating, and discussion website.  At peak times, Reddit can see over 300,000 comments and 35,000 submissions an hour.  The Reddit API offers developers a simple way to collect all of this data. AWS offers Amazon S3 to store data, Amazon Kinesis Firehose to stream data, AWS Glue to catalog data, and Amazon Athena to analyze data. TextBlob is a Python (2 and 3) library for processing textual data. It provides a simple API for diving into common natural language processing (NLP) tasks such as part-of-speech tagging, noun phrase extraction, sentiment analysis, classification, translation, and more.

Put this all together and you can deploy a streaming data lake architecture that collects, analyzes, and stores reddit comments in near realtime. Finally, the data can be queried via Amazon Athena.

### Example of a Collected+Analyzed Reddit Comment:

        {
            "comment_id": "fx3wgci",
            "subreddit": "Fitness",
            "author": "silverbird666",
            "comment_text": "well, i dont exactly count my calories, but i run on a competitive base and do kickboxing, that stuff burns quite much calories. i just stick to my established diet, and supplement with protein bars and shakes whenever i fail to hit my daily intake of protein. works for me.",
            "distinguished": null,
            "submitter": false,
            "total_words": 50,
            "reading_ease_score": 71.44,
            "reading_ease": "standard",
            "reading_grade_level": "7th and 8th grade",
            "sentiment_score": -0.17,
            "censored": 0,
            "comment_language": "en",
            "positive": 0,
            "neutral": 1,
            "negative": 0,
            "subjectivity_score": 0.35,
            "subjective": 0,
            "url": "https://reddit.com/r/Fitness/comments/hlk84h/victory_sunday/fx3wgci/",
            "comment_date": "2020-07-06 15:41:15",
            "comment_timestamp": "2020/07/06 15:41:15",
            "comment_hour": 15,
            "comment_year": 2020,
            "comment_month": 7,
            "comment_day": 6
        }

The Python application can be viewed here: https://github.com/CYarros10/reddit-streaming-application

----



## Tutorial

### 1. Create your Reddit bot account

1. [Register a reddit account](https://www.reddit.com/register/)

2. Follow prompts to create new reddit account:
    * Provide email address
    * Choose username and password
    * Click Finish

3. Once your account is created, go to [reddit developer console.](https://www.reddit.com/prefs/apps/)

4. Select **“are you a developer? Create an app...”**

5. Give it a name.

6. Select script.  <--- **This is important!**

7. For about url and redirect uri, use http://127.0.0.1

8. You will now get a client_id (underneath web app) and secret

9. Keep track of your Reddit account username, password, app client_id (in blue box), and app secret (in red box). These will be used in tutorial Step 11

#### Further Learning / References: PRAW

* [PRAW Quick start](https://praw.readthedocs.io/en/latest/getting_started/quick_start.html)

----

### 2. Create a Key Pair for your streaming server

1. Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/) or select EC2 under Services dropdown

2. In the navigation pane, under NETWORK & SECURITY, choose Key Pairs

    Note: The navigation pane is on the left side of the Amazon EC2 console. If you do not see the pane, it might be minimized; choose the arrow to expand the pane

3. Choose Create Key Pair

4. For Key pair name, enter a name for the new key pair (ex: RedditBotKey), and then choose Create

5. The private key file is automatically downloaded by your browser. The base file name is the name you specified as the name of your key pair, and the file name extension is .pem. Save the private key file in a safe place

    Important: This is the only chance for you to save the private key file. You'll need to provide the name of your key pair when you launch an instance and the corresponding private key each time you connect to the instance

#### Further Learning / References: EC2 Key Pairs

* [User Guide: EC2 Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

----

### 3. Deploy streaming datalake infrastructure in CloudFormation

1. Open the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation) or select CloudFormation under the Services dropdown

2. Click Create New Stack / Create Stack

3. In the Template section, select Upload a template file

4. Select Choose File and upload templates/cloudformation.yaml

5. Click Next

6. Enter Parameters, including your reddit client/secret info created earlier.

7. Continue to Create

----

### 4. Wait 10 minutes ...

----

### 5. Use Athena to develop insights

1. Open the [Amazon Athena console](https://console.aws.amazon.com/athena/) or select Athena in the Services dropdown

2. Choose the glue database (reddit_db) populated on the left view

3. Select the table (reddit_comments) to view the table schema

4. You should now be able to use SQL to query the table (S3 data)

Examples:

        -- total number of comments

        select count(*)
        from reddit_comments;


        -- general sentiment of reddit on a day

        select round(avg(sentiment_score), 4) as avg_sentiment
        from reddit_comments
        where comment_date
        like '%2020-07-06%';


        -- total comments collected per subreddits

        select count(*) as num_comments, subreddit
        from reddit_comments
        group by subreddit
        order by num_comments DESC;


        -- average sentiment per subreddits

        select round(avg(sentiment_score), 4) as avg_sentiment_score, subreddit
        from reddit_comments
        group by subreddit
        order by avg_sentiment_score DESC;


        -- list all Subreddits

        select distinct(subreddit)
        from reddit_comments;


        -- top 10 most positive comments by subreddit

        select subreddit, comment_text
        from reddit_comments
        where subreddit = 'aww'
        order by sentiment_score DESC
        limit 10;


        -- most active subreddits and their sentiment

        select subreddit, count(*) as num_comments, round(avg(sentiment_score), 4) as avg_sentiment_score
        from reddit_comments
        group by subreddit
        order by num_comments DESC;


        -- search term frequency by subreddit where comments greater than 5

        select subreddit, count(*) as comment_occurrences
        from reddit_comments
        where LOWER(comment_text) like '%hey%'
        group by subreddit
        having count(*) > 5
        order by comment_occurrences desc;


        -- search term sentiment by subreddit

        select subreddit, round(avg(sentiment_score), 4) as avg_sentiment_score
        from reddit_comments
        where LOWER(comment_text) like '%puppy%'
        group by subreddit
        having count(*) > 5
        order by avg_sentiment_score desc;


        -- top 25 most positive comments about a search term

        select subreddit, author, comment_text, sentiment_score
        from reddit_comments
        where LOWER(comment_text) like '%puppy%'
        order by sentiment_score desc
        limit 25;

#### Further Learning / References: Athena and Glue

* [Using Glue and Athena](https://docs.aws.amazon.com/athena/latest/ug/glue-athena.html)

#### Next Steps: Terminate Services (if you're done)

* Hopefully by now you have found some interesting insights into Reddit and the overall public sentiment.  Athena is a great service for ad-hoc queries like this.  You are approaching the end of this tutorial, so you will start terminating services and instances to prevent further billing.

----

### Clean up the environment

# Do not skip this step! Leaving AWS resources without tearing down can result a bill in the end of the month. Make sure you follow the steps to remove the resources you’ve created

1. Delete the data in your reddit datalake bucket

2. Our infrastructure was created from a CloudFormation template, we’ll delete the stack and the key pair

    * Open the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation) or select CloudFormation under Services dropdown
    * On the left panel that says stacks – click on the EC2 stack you’ve created
    * Click Delete on top of the pane
    * Open the [Amazon EC2 console](https://console.aws.amazon.com/ec2/) or select EC2 under Services dropdown
    * In the navigation pane, under NETWORK & SECURITY, choose Key Pairs
    * Click on the key pair name you’ve created and hit Delete on top
