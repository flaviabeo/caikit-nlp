# Set up and run locally caikit embeddings server

#### Setting Up Virtual Environment using Python venv

For [(venv)](https://docs.python.org/3/library/venv.html), make sure you are in an activated `venv` when running `python` in the example commands that follow. Use `deactivate` if you want to exit the `venv`.

```shell
python3 -m venv venv
source venv/bin/activate
```

### Models

To create a model configuration and artifacts, the best practice is to run the module's bootstrap() and save() methods.  This will:

* Load the model by name (from Hugging Face hub or repository) or from a local directory. The model is loaded using the sentence-transformers library.
* Save a config.yml which:
  * Ties the model to the module (with a module_id GUID)
  * Sets the artifacts_path to the default "artifacts" subdirectory
  * Saves the model in the artifacts subdirectory

This can be done by running the `boostrap_model.py` script in your virtual environment.

```shell
source venv/bin/activate
./demo/server/bootstrap_model.py -m <MODEL_NAME_OR_PATH> -o <OUTPUT_DIR>
```


To avoid overwriting your files, the save() will return an error if the output directory already exists. You may want to use a temporary name. After success, move the output directory to a `<model-id>` directory under your local models dir.


### Starting the Caikit Runtime

Run caikit-runtime configured to use the caikit-nlp library. Set up the following environment variables:

```bash
export RUNTIME_HTTP_ENABLED=true
export RUNTIME_LOCAL_MODELS_DIR=/models
export RUNTIME_LAZY_LOAD_LOCAL_MODELS=true
```

In one terminal, start the runtime server:

```bash
source venv/bin/activate
pip install -r requirements.txt
caikit-runtime
```

### Embedding retrieval example Python client

In another terminal, run the example client code to retrieve embeddings.

```shell
source venv/bin/activate
cd demo/client
MODEL=<model-id> python embeddings.py
```

The client code calls the model and queries for embeddings using 2 example sentences.

You should see output similar to the following:

```ShellSession
$ python embeddings.py
INPUT TEXTS:  ['test first sentence', 'another test sentence']
OUTPUT: {
  {
  "results": [
    [
      -0.17895537614822388,
      0.03200146183371544,
      -0.030327674001455307,
      ...
    ],
    [
      -0.17895537614822388,
      0.03200146183371544,
      -0.030327674001455307,
      ...
    ]
  ],
  "producerId": {
    "name": "EmbeddingModule",
    "version": "0.0.1"
  },
  "inputTokenCount": "9"
  }
}
LENGTH:  2  x  384
```

### Sentence similarity example Python client

In another terminal, run the client code to infer sentence similarity.

```shell
source venv/bin/activate
cd demo/client
MODEL=<model-id> python sentence_similarity.py
```

The client code calls the model and queries sentence similarity using 1 source sentence and 2 other sentences (hardcoded in sentence_similarity.py). The result produces the cosine similarity score by comparing the source sentence with each of the other sentences.

You should see output similar to the following:

```ShellSession
$ python sentence_similarity.py   
SOURCE SENTENCE:  first sentence
SENTENCES:  ['test first sentence', 'another test sentence']
OUTPUT: {
  "result": {
    "scores": [
      1.0000001192092896
    ]
  },
  "producerId": {
    "name": "EmbeddingModule",
    "version": "0.0.1"
  },
  "inputTokenCount": "9"
}
```

### Reranker example Python client

In another terminal, run the client code to execute the reranker task using both gRPC and REST.

```shell
source venv/bin/activate
cd demo/client
MODEL=<model-id> python reranker.py
```

You should see output similar to the following:

```ShellSession
$ python reranker.py
======================
TOP N:  3
QUERIES:  ['first sentence', 'any sentence']
DOCUMENTS:  [{'text': 'first sentence', 'title': 'first title'}, {'_text': 'another sentence', 'more': 'more attributes here'}, {'text': 'a doc with a nested metadata', 'meta': {'foo': 'bar', 'i': 999, 'f': 12.34}}]
======================
RESPONSE from gRPC:
===
QUERY:  first sentence
  score: 0.9999997019767761  index: 0  text: first sentence
  score: 0.7350112199783325  index: 1  text: another sentence
  score: 0.10398174077272415  index: 2  text: a doc with a nested metadata
===
QUERY:  any sentence
  score: 0.6631797552108765  index: 0  text: first sentence
  score: 0.6505964398384094  index: 1  text: another sentence
  score: 0.11903437972068787  index: 2  text: a doc with a nested metadata
===================
RESPONSE from HTTP:
{
    "results": [
        {
            "query": "first sentence",
            "scores": [
                {
                    "document": {
                        "text": "first sentence",
                        "title": "first title"
                    },
                    "index": 0,
                    "score": 0.9999997019767761,
                    "text": "first sentence"
                },
                {
                    "document": {
                        "_text": "another sentence",
                        "more": "more attributes here"
                    },
                    "index": 1,
                    "score": 0.7350112199783325,
                    "text": "another sentence"
                },
                {
                    "document": {
                        "text": "a doc with a nested metadata",
                        "meta": {
                            "foo": "bar",
                            "i": 999,
                            "f": 12.34
                        }
                    },
                    "index": 2,
                    "score": 0.10398174077272415,
                    "text": "a doc with a nested metadata"
                }
            ]
        },
        {
            "query": "any sentence",
            "scores": [
                {
                    "document": {
                        "text": "first sentence",
                        "title": "first title"
                    },
                    "index": 0,
                    "score": 0.6631797552108765,
                    "text": "first sentence"
                },
                {
                    "document": {
                        "_text": "another sentence",
                        "more": "more attributes here"
                    },
                    "index": 1,
                    "score": 0.6505964398384094,
                    "text": "another sentence"
                },
                {
                    "document": {
                        "text": "a doc with a nested metadata",
                        "meta": {
                            "foo": "bar",
                            "i": 999,
                            "f": 12.34
                        }
                    },
                    "index": 2,
                    "score": 0.11903437972068787,
                    "text": "a doc with a nested metadata"
                }
            ]
        }
    ],
     "producerId": {
    "name": "EmbeddingModule",
    "version": "0.0.1"
    },
    "inputTokenCount": "9"
}
```