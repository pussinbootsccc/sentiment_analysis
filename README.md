# sentiment_analysis

The software implements sentiment analysis algorithms at two granularities. 

Note the meaning of a document depends on the context. 
For interview analysis, a document is corresponding to a piece of note;
For survey analysis, a document is obtained by collapsing all responses by a particular respondent (for that survey). 

### Global Analysis

*Input*:
* `projectId`: A positive integer, i.e. Project ID of interest.
* `surveyId`: A positive integer, i.e. Survey ID of interest. The argument will have no effect when `table == interview` or `table == both` (thus is recommended to be set to `0` in those cases).
* `table`: A string-valued variable taking values among either `interview`, `survey` or `both`. 

*Output*: A list of automatically extracted keywords from a set of documents. The documents are chosen based on the specified constraints——when `projectId = 123`, `surveyId=456` and `table=survey`, only question answers in survey 456 under project 123 will be used for keywords selection.

Each keyword is represented as a Python object containing the following fields:
* `name`: Name of the keyword.
* `allPercentage`: Proportion of docs containing the keyword.
* `negPercentage`: Proportion of docs with negative sentiment containing the keyword.
* `posPercentage`: Proportion of docs with positive sentiment containing the keyword.
* `isFromInterview`: Takes value `1` if the keyword is present in the interview table; takes value `0` otherwise.
* `isFromSurvey`: Takes value `1` if the keyword is present in the survey table; takes value `0` otherwise.

### Detailed Analysis

*Input*:
same as for global analysis. 

*Output*: A list of document-level meta-information corresponding to each doc. The docs are selected based on the specified constraints per input.

Each meta-info is represented as a Python object containing the following fields:
* `keywrods`: A list of keywords that appeared in the doc. This is the intersection of the keywords identified by global analysis and the keywords appeared in the doc. Each keyword here is an object containing two fields:
  - `name`: Name of the keyword.
  - `score`: Sentiment score of the keyword in the given document (context). 
* `score`: The overall sentiment score for the doc (details described below). A positive score indicates a positive overall sentiment of the document, and vice versa.
* `respondentId`: Id of the current document. The field will be equal to `respondentId` if `table == survey`, and equal to `id` if `table == interview`.

## Implementation Details

##### How the docs are preprocessed:
We split each doc per sentence, and tokenize each sentence into words. Stemming is performed using the Porter stemmer provided by `Python`'s natural language toolkit (`NLTK`). Common English stop words were removed from the token list.

##### How global keywords are extracted
Top 128 keywords with the highest TF-IDF are chosen. TF stands for term-frequency and IDF stands for document-frequency. This is a standard approach for extracting the most informative vocabularies. 

##### How `allPercentage` is computed:
```
allPercentage = #(docs containing the keyword) / #(docs)
```

##### How `posPercentage (negPercentage)` is computed:
```
posPercentage = #(docs with positive sentiment containing the keyword) / #(docs)
```

##### How do we score a doc:
By aggregating the sentiment scores of its sentences. The sentiment score for a sentence is obtained by aggregating the token scores, normalized by the sentence length `#(tokens in the sentence)`. Token scores are defined in `*.yml` files. We let `score = +1` for positive tokens and `score = -1` for the negative ones. 

##### How do we score a keyword for a given doc:
By aggregating the sentiment scores of sentences where the keyword appears in. 

## Software Dependencies:

Carrying out the sentiment analysis requires Python 2.7 with the NLTK package installed, and a set of external `*.yml` files containing predefined sentiment of some seed keywords. Jython is also required to allow the python scripts to interact with the Java controller and the `mySQL` backend. More details about how Jython communicates with the database backend can be found at the definition of class `DBHander` in `utils/DataHandler.py`.
