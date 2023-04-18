# vector-databases
Simple examples for vector databse

# weaviate Node Js App


```
const weaviate = require('weaviate-ts-client');

// Instantiate the client with the auth config
const client = weaviate.client({
  scheme: 'https',
  host: 'mysandbox-jsi3dlvm.weaviate.network',
  apiKey: new weaviate.ApiKey(''),
  headers: {'X-Cohere-Api-Key': ''},  // Replace w/ your API Key for the Weaviate instance
});

let classObj = {
    'class': 'Question',
    'vectorizer': 'text2vec-cohere'
}

// add the schema
client
  .schema
  .classCreator()
  .withClass(classObj)
  .do()
  .then(res => {
    console.log(res)
  })
  .catch(err => {
    console.error(err)
  });
  
  async function getJsonData() {
    const file = await fetch('https://raw.githubusercontent.com/weaviate-tutorials/quickstart/main/data/jeopardy_tiny.json');
    return file.json();
}

async function importQuestions() {
  // Get the data from the data.json file
  const data = await getJsonData();

  // Prepare a batcher
  let batcher = client.batch.objectsBatcher();
  let counter = 0;
  let batchSize = 100;

  data.forEach(question => {
    // Construct an object with a class and properties 'answer' and 'question'
    const obj = {
      class: 'Question',
      properties: {
        answer: question.Answer,
        question: question.Question,
        category: question.Category,
      },
    }

    // add the object to the batch queue
    batcher = batcher.withObject(obj);

    // When the batch counter reaches batchSize, push the objects to Weaviate
    if (counter++ == batchSize) {
      // flush the batch queue
      batcher
      .do()
      .then(res => {
        console.log(res)
      })
      .catch(err => {
        console.error(err)
      });

      // restart the batch queue
      counter = 0;
      batcher = client.batch.objectsBatcher();
    }
  });

  // Flush the remaining objects
  batcher
  .do()
  .then(res => {
    console.log(res)
  })
  .catch(err => {
    console.error(err)
  });
}

importQuestions();
  client.graphql
  .get()
  .withClassName('Question')
  .withFields('question answer category')
  .withNearText({concepts: ['biology']})
  .withLimit(2)
  .do()
  .then(res => {
    console.log(JSON.stringify(res, null, 2))
  })
  .catch(err => {
    console.error(err)
  });
```


# weaviate Python App

```

import weaviate
import json

auth_config = weaviate.auth.AuthApiKey(
  api_key=""
)  # Replace w/ your API Key for the Weaviate instance

client = weaviate.Client(
  url=
  "https://mysandbox-jsi3dlvm.weaviate.network",  # Replace with your endpoint
  auth_client_secret=auth_config,
  additional_headers={
    'X-OpenAI-Api-Key':
    'sk-'  # Or "X-Cohere-Api-Key" or "X-HuggingFace-Api-Key"
  })

# ===== import data =====
# Load data
import requests

url = 'https://raw.githubusercontent.com/weaviate-tutorials/quickstart/main/data/jeopardy_tiny.json'
resp = requests.get(url)
data = json.loads(resp.text)

# Prepare a batch process
with client.batch as batch:
  batch.batch_size = 100
  # Batch import all Questions
  for i, d in enumerate(data):
    # print(f"importing question: {i+1}")  # To see imports

    properties = {
      "answer": d["Answer"],
      "question": d["Question"],
      "category": d["Category"],
    }

    client.batch.add_data_object(properties, "Question")

some_objects = client.data_object.get()
print(json.dumps(some_objects))

```



