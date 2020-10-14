# Elastic-Search
The goal of this project is to become familiar with the process of installing and starting Elasticsearch. Elasticsearch is an information retrieval system that can index documents as we saw in class, and can respond to queries. This assignment will help you begin to understand how modern search engines work, and how documents and queries can be represented for information retrieval tasks.

The attached file test.json contains Reddit comments from December 2006, available by the https://pushshift.io open data initiative.

**Setup**
- Make sure you have Java JDK 8 installed. This project is tested with java version 1.8.0_191. (Check by executing the command java -version in a terminal window.) You should see something like

    java version "1.8.0_191"
    
    Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
    
    Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)

    Java 9 is not recommended. It might cause some problems. Java can be downloaded at              
    
    https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html.

- Install Elasticsearch

    - On a Mac, Elasticsearch is most easily installed with “HomeBrew”

        - Ensure command line tools are installed by running xcode-select --install in a terminal
        - Homebrew can be installed by running the following command in a terminal:
        - /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
        - Follow the instructions. On completing, run
        - brew update
        - brew tap elastic/tap
        - brew install elasticsearch
        - brew install elastic/tap/elasticsearch-full
        - Open terminal window and run elasticsearch. You will see a series of messages as it starts up. Keep this window open.

    - On Windows

        - Follow the instructions here, installing elasticsearch “as a service” (You will see the option.)      
        
         https://www.elastic.co/guide/en/elasticsearch/reference/current/windows.html
         
        - To install curl, first install https://chocolatey.org/. Note that you can open a Command Prompt (Admin) by choosing it            from the menu that appears if you hold the Windows key and press X.
        
        - Then into a new Command Prompt (Admin), menu type choco install curl

    - On a Linux or Linux-like environment**

        - Follow the instructions here https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html

If you have correctly installed everything and Elasticsearch is running, clicking this link http://localhost:9200 should show you a JSON object similar to

{
  "name" : "dlizotte.wks.csd.uwo.ca",
  "cluster_name" : "elasticsearch_dlizotte",
  "cluster_uuid" : "6lor8wPIRe-XUwUM_efyQg",
  "version" : {
    "number" : "7.5.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "3ae9ac9a93c95bd0cdc054951cf95d88e1e18d96",
    "build_date" : "2019-12-16T22:57:37.835892Z",
    "build_snapshot" : false,
    "lucene_version" : "8.3.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}


**Running Elasticsearch Commands**

Elasticsearch uses a client-server model of operation. For this assignment, you will use two terminal windows. In one, you will run the elasticsearch server. In the other, you will run commands that access the server.

To start the server on a Mac, open a terminal window and run the command elasticsearch. You can terminate the server when you are finished by pressing ctrl-c in that window. On Linux or Windows, follow the instructions above under Setup.

For this project, we will use curl, which is a command-line program that communicates with the elasticnet server. The tasks below will involve running curl by copying and pasting commands into a terminal window (Mac or Linux) or a Command Prompt window (Windows). Before you start, ensure you have the elasticsearch server running in a terminal window (MacOS) or start the service running as described above (Linux, Windows).


**Questions:**

**Indexing**

- Basic Indexing

1) Follow along with the Getting Started Guide Exploring Your Cluster chapter https://www.elastic.co/guide/en/elasticsearch/reference/6.7/getting-started-explore.html, from the Cluster Health up to and including the Delete an Index section of the chapter. For each command in the Starting Guide, click the Copy as cURL link and paste the command into your command window. Run the command, and copy the command and its output into a text file called elasticcommands.txt for submission. Note that we want you to follow along with the old version (6.7) of the guide, but it will work with the newest version of elastic search.

2) Why we might break an index into shards.

3) Why we might replicate an index.

4) Why your cluster health is yellow.

- Indexing Reddit

1) Ensure that your Elasticsearch instance has no indices in it. Provide the command you used to verify this.

2) Download the test.json file from the OWL assignment page. Run the following command, ensuring that you do this from the same folder that has the test.json file in it. What is the name of the index created by the command?

curl -s -H 'Content-Type: application/json' -XPOST localhost:9200/_bulk --data-binary @test.json

3) How large is the index? (pri.store.size)

**Search**

Answer the following questions using the three example search queries below.

1) Do the three queries below return different sets of documents? How can you tell?

2) Give the highest score for each query.

3) What can you deduce about the default stemming procedure used by Elasticsearch?

4) Give a query that could be used to check whether or not Elasticsearch removes a common English stopword.

Query 1

curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{ "query": { "match" : { "body" : { "query" : "cat"} } } } '

Query 2

curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{ "query": { "match" : { "body" : { "query" : "cats"} } } } '

Query 3

curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{ "query": { "match" : { "body" : { "query" : "cat cats", "operator" : "and" } } } }'

**Analyzers**

Elasticsearch uses what it calls “analyzers” to process text data when indexing. https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html

1) Delete the all indices in your Elasticsearch instance before beginning this section. Submit the code you use to do so.

2) To change how Elasticsearch indexes documents, we provide settings for an index before documents are added to it. (This is why we had to delete all indices before running these next commands.) Run the following two (quite large) commands to change the default stemming behaviour. The first defines a new analyzer, my_analyzer, which uses a stemmer. It creates a comments index and adds this analyzer to it. The second informs Elasticsearch that it should use my_analyzer when analyzing the body field of documents going into the comments index.

curl -XPUT 'localhost:9200/comments?pretty' -H 'Content-Type: application/json' -d'
{
    "settings": {
        "analysis" : {
            "analyzer" : {
                "my_analyzer" : {
                    "tokenizer" : "standard",
                    "filter" : ["lowercase", "my_stemmer"]
                }
            },
            "filter" : {
                "my_stemmer" : {
                    "type" : "stemmer",
                    "name" : "english"
                }
            }
        }
    }
}'
curl -XPUT 'localhost:9200/comments/_mapping?pretty' -H 'Content-Type: application/json' -d'
{
        "properties" : {
          "body" : {
            "type" : "text",
            "analyzer" : "my_analyzer"
          }
        }
}
'
You can test your analyzer on a specific piece of text like so:

curl -X POST "localhost:9200/comments/_analyze?pretty" -H 'Content-Type: application/json' -d'
{
  "analyzer": "my_analyzer",
  "text": "I\u0027m a :) person, and you?"
}
'

Since you would have deleted your index as specified earlier in the section, before you answer the next questions you must re-index the documents by running this command again: curl -s -H 'Content-Type: application/json' -XPOST localhost:9200/_bulk --data-binary @test.json

Re-run the three queries from the previous section. Describe how the results have changed.
Has the size of the index become larger or smaller compared with the previous runs? Why do you think this might be?

The answers to the questions above can be found in "Answers.pdf".
For the indexing section, check out "elasticcommands.txt" file.
