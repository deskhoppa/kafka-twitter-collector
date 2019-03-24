# DeskHoppa - Kafka, KSQL and Twitter Implementation

Finding out what the public, your prospective customers, is now a default starting point. Twitter gives us a good starting point and the use of hashtags gives us an easy way to filter out tweets and refine what we're looking for.

In times gone by it was a coding job to get the Twitter API to get the data needed. Now there are easier ways to do it. The excellent post on the Confluent website by Robin Moffatt called [Getting Started Analyzing Twitter Data in Apache Kafka through KSQL](https://www.confluent.io/blog/using-ksql-to-analyse-query-and-transform-data-in-kafka) gives us a framework to easily pull Twitter data via Kafka and the KSQL engine. All that's required is some configuration and some thought about what information you want.

Just to note, none of the actual code or content here is new. It's just packaged and shared here for two reasons, firstly [DeskHoppa](https://www.deskhoppa.com) has a quick grab repo to get up and running quickly and secondly that it might be handy to someone else.

Obviously we do a lot more with machine learning in the processing of the data but this first step gives us a great, simple way to get the initial work done quickly and efficiently. We extend our thanks to everyone who wrote it.

# Setup

The `custom`, `etc` and `share` directories in the repository mirror those in the Confluent implementation.

Copy the contents of respective directories in this repository to your Kafka distribution.

## Twitter Connect Setup

In the `share/custom` directory is the JSON configuration for the Kafka Connect profile.

You will need to have a Twitter Application setup from the [Twitter Developer Website](https://developer.twitter.com/). The access, secret and consumer keys are required in the twitter-source.json file. As for what you want to track from the Twitter stream, add those to the `filter.keywords` key (we left a couple of examples, you might want to replace them).


```
{
 "name": "twitter_source_json_01",
 "config": {
   "connector.class": "com.github.jcustenborder.kafka.connect.twitter.TwitterSourceConnector",
   "twitter.oauth.accessToken": "",
   "twitter.oauth.consumerSecret": "",
   "twitter.oauth.consumerKey": "",
   "twitter.oauth.accessTokenSecret": "",
   "kafka.delete.topic": "twitter_deletes_json_01",
   "value.converter": "org.apache.kafka.connect.json.JsonConverter",
   "key.converter": "org.apache.kafka.connect.json.JsonConverter",
   "value.converter.schemas.enable": false,
   "key.converter.schemas.enable": false,
   "kafka.status.topic": "twitter_json_01",
   "process.deletes": true,
   "filter.keywords": "#ahashtag, #anotherhashtag"
 }
}
```

In the `etc/kafka-connect-twitter` directory the `connect-avro-docker.properties` file has a `plugin.path` setting, in this repository it's setup as a relative path to the Confluent directory structure. You may want to change this to your setup.

## Running The Connect Setup

Assuming that Zookeeper and Kafka are running it's a simple case of using Connect to install the JSON configuration file.

```
confluent load twitter_source -d /path/to/your/file/twitter-source.json
```

If all is well then Twitter data should be sent to the topic. Using the handy `kafka-console-consumer` script in the `bin` directory you can see the raw tweets coming in.

# KSQL

In the `ksql-queries` are two ksql queries that are helpful.

- `create_twitter_raw.ksql` Do this first, creates the stream of raw Twitter data. The query stream below reads from this stream.
- `create_stream.ksql` Creates a table stream of cleaned up Twitter data. Note the use of the `EXTRACTJSONFIELD` function, rather handy.
- `reset_offset.ksql` While testing it's good to be able to reset the topic offset back to the start so you can retest your queries.
