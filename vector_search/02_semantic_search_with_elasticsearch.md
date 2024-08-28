# Semantic Search with Elasticsearch
Elasticsearch, is a powerful open-source search engine, that has been used for full-text search, analytics, and data visualization. Traditionally, Elasticsearch has been known for its ability to perform keyword-based search, where documents are matched based on exact or partial keyword matches. However, Elasticsearch has evolved to support semantic search—an approach that focuses on understanding the meaning behind words and phrases, rather than just matching keywords.

Semantic search in Elasticsearch enables more intuitive and context-aware search experiences, making it possible to find relevant information even when the exact keywords are not present in the query. This article will explore how semantic search can be implemented in Elasticsearch, its benefits, and practical use cases.

## Semantic Search
Semantic search is a technique that goes beyond traditional keyword-based search by considering the context, intent, and meaning of the search query. Unlike keyword search, which focuses on literal matches, semantic search understands the relationships between words and concepts, enabling more accurate and relevant search results.

For example, in a keyword-based search, a query like "laptop battery life" might return documents containing those exact words. However, a semantic search could return documents that discuss related concepts like "long-lasting laptops," "energy-efficient devices," or even "portable computers with extended battery life."

## How Elasticsearch Supports Semantic Search
Elasticsearch supports semantic search through a combination of techniques, including:

* Vector Representations: Using pre-trained models like BERT (Bidirectional Encoder Representations from Transformers) to convert text into dense vector embeddings that capture semantic meaning.

* Similarity Scoring: Measuring the similarity between query vectors and document vectors to rank search results based on semantic relevance.

* Custom Analyzers: Creating custom tokenizers, filters, and analyzers that preprocess text data in ways that enhance semantic understanding.

* Integration with Machine Learning Models: Leveraging machine learning models that are integrated with Elasticsearch to perform tasks like entity recognition, sentiment analysis, and more

## Architecture - Semantic Search using Elastic Search
![Architecture Semantic search using Elasticsearch](./semanticsearch_with_elasticsearch.png)

Two very important concepts in Elasticsearch are documents and indexes.

## Documents
A document is a collection of fields with their associated values. Each document is a JSON object that contains data in a structured format. For example, a document representing a book might contain fields like title, author, and publication date.

## Indexes
An index is a collection of documents that is stored in a highly optimized format designed to perform efficient searches. Indexes are similar to tables in a relational database, but they are more flexible and can store complex data structures.

To work with Elasticsearch, you need to organize your data into documents and then add all your documents to an index. This allows Elasticsearch to efficiently search and retrieve relevant documents based on the search queries.


# Implementing Semantic Search in Elasticsearch
## 1. Setting Up Elasticsearch
First, ensure that you have Elasticsearch up and running.You can do this by starting a Docker container for Elasticsearch, to connect to elasticsearch remotely:
```
docker run -it \
    --rm \
    --name elasticsearch \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.4.3

```

## 2 Data Loading and Preprocessing
In this step, we will load the documents.json file and preprocess it to flatten the hierarchy, making it suitable for Elasticsearch. The documents.json file contains a list of courses, each with a list of documents. We will extract each document and add a course field to it, indicating which course it belongs to.
```
import json

with open('documents.json', 'rt') as f_in:
    docs_raw = json.load(f_in)

```
Elasticsearch requires to have everything on the same level of hierachry, in this case we have two levels, course and documents
```
documents = []

for course_dict in docs_raw:
    for doc in course_dict['documents']:
        doc['course'] = course_dict['course']
        documents.append(doc)

documents[1]
#Output
{'text': 'GitHub - DataTalksClub data-engineering-zoomcamp#prerequisites',
 'section': 'General course-related questions',
 'question': 'Course - What are the prerequisites for this course?',
 'course': 'data-engineering-zoomcamp'}

```
## 3 Create Embeddings Using Pretrained Models
To perform semantic search, we need to convert our documents into dense vectors (embeddings) that capture the semantic meaning of the text. We will use a pre-trained model from the Sentence Transformers library to generate these embeddings. These embeddings are then indexed into Elasticsearch. These embeddings enable us to perform semantic search, where the goal is to find text that is contextually similar to a given query.

The text and question fields are the actual data fields containing the primary information, whereas other fields like section and course are more categorical and less informative for the purpose of creating meaningful embeddings.

* Install the sentence_transformers library.
* Load the pre-trained model and use it to generate embeddings for our documents.
```
from sentence_transformers import SentenceTransformer
model = SentenceTransformer("all-mpnet-base-v2")

#created the dense vector using the pre-trained model
operations = []
for doc in documents:
    # Transforming the title into an embedding using the model
    doc["text_vector"] = model.encode(doc["text"]).tolist()
    operations.append(doc)


```
## 4 Connecting to Elasticsearch
In this step, we will set up a connection to an Elasticsearch instance. Make sure you have Elasticsearch running.
```
from elasticsearch import Elasticsearch

# Connect to the Elasticsearch instance
es_client = Elasticsearch('http://localhost:9200')

# Check the connection
print(es_client.info())
```
