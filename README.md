# 검색 어플리케이션 만들기

[![Introduction to Generative AI and Large Language Models](./images/08-lesson-banner.png?WT.mc_id=academic-105485-koreyst)](TBD)

> **Video Coming Soon**

LLM은 챗봇과 텍스트 생성에만 국한되지 않는다. 임베딩을 활용해 검색 애플리케이션을 만드는 것도 가능하다. 임베딩은 데이터를 수치적으로 표현한 벡터로, 데이터의 의미론적 검색에 활용될 수 있다.

In this lesson, you are going to build a search application for our education startup. Our startup is a non-profit organization that provides free education to students in developing countries. Our startup has a large number of YouTube videos that students can use to learn about AI. Our startup wants to build a search application that allows students to search for a YouTube video by typing a question.

예를 들어, 학생이 '주피터 노트북이 무엇인가요?' 또는 'Azure ML이 무엇인가요?'와 같은 질문을 입력하면, 검색 애플리케이션은 해당 질문과 관련된 유튜브 동영상 목록을 반환한다. 더 좋은 점은, 검색 애플리케이션이 질문의 답이 있는 동영상의 해당 부분으로 연결되는 링크를 제공한다는 것이다.

## 도입

이 수업에서 다룰 내용은 다음과 같다:

- 의미론적 검색과 키워드 검색의 차이.
- 텍스트 임베딩이란 무엇인가.
- 텍스트 임베딩 인덱스 생성하기.
- 텍스트 임베딩 인덱스 검색하기.

## 학습 목표

이 수업을 마치고 나면, 여러분은 다음과 같은 작업을 할 수 있게 된다:

- 의미론적 검색과 키워드 검색의 차이를 설명할 수 있다. 
- 텍스트 임베딩이 무엇인지 설명할 수 있다. 
- 임베딩을 사용하여 데이터를 검색하는 애플리케이션을 만들 수 있다.

## Why build a search application?

Creating a search application will help you understand how to use Embeddings to search for data. You will also learn how to build a search application that can be used by students to find information quickly.

The lesson includes an Embedding Index of the YouTube transcripts for the Microsoft [AI Show](https://www.youtube.com/playlist?list=PLlrxD0HtieHi0mwteKBOfEeOYf0LJU4O1?WT.mc_id=academic-105485-koreyst) YouTube channel. The AI Show is a YouTube channel that teaches you about AI and machine learning. The Embedding Index contains the Embeddings for each of the YouTube transcripts up until Oct 2023. You will use the Embedding Index to build a search application for our startup. The search application returns a link to the place in the video where the answer to the question is located. This is a great way for students to find the information they need quickly.

The following is an example of a semantic query for the question 'can you use rstudio with azure ml?'. Check out the YouTube url, you'll see the url contains a timestamp that takes you to the place in the video where the answer to the question is located.

![Semantic query for the question "can you use rstudio with Azure ML"](./images/query-results.png?WT.mc_id=academic-105485-koreyst)

## What is semantic search?

Now you might be wondering, what is semantic search? Semantic search is a search technique that uses the semantics, or meaning, of the words in a query to return relevant results.

Here is an example of a semantic search. Let's say you were looking to buy a car, you might search for 'my dream car', semantic search understands that you are not `dreaming` about a car, but rather you are looking to buy your `ideal` car. Semantic search understands your intention and returns relevant results. The alternative is `keyword search` which would literally search for dreams about cars and often returns irrelevant results.

## What are Text Embeddings?

[Text embeddings](https://en.wikipedia.org/wiki/Word_embedding?WT.mc_id=academic-105485-koreyst) are a text representation technique used in [natural language processing](https://en.wikipedia.org/wiki/Natural_language_processing?WT.mc_id=academic-105485-koreyst). Text embeddings are semantic numerical representations of text. Embeddings are used to represent data in a way that is easy for a machine to understand.  There are many models for building text embeddings, in this lesson, we will focus on generating embeddings using the OpenAI Embedding Model.

Here's an example, imagine the following text is in a transcript from one of the episodes on the AI Show YouTube channel:

```text
Today we are going to learn about Azure Machine Learning.
```

We'd pass the text to the OpenAI Embedding API and it would return the following embedding consisting of 1536 numbers aka a vector. Each number in the vector represents a different aspect of the text. For brevity, here are the first 10 numbers in the vector.

```python
[-0.006655829958617687, 0.0026128944009542465, 0.008792596869170666, -0.02446001023054123, -0.008540431968867779, 0.022071078419685364, -0.010703742504119873, 0.003311325330287218, -0.011632772162556648, -0.02187200076878071, ...]
```

## How is the Embedding index created?

The Embedding index for this lesson was created with a series of Python scripts. You'll find the scripts along with instructions in the [README](./scripts/README.md?WT.mc_id=academic-105485-koreyst) in the 'scripts` folder for this lesson. You don't need to run these scripts to complete this lesson as the Embedding Index is provided for you.

The scripts perform the following operations:

1. The transcript for each YouTube video in the [AI Show](https://www.youtube.com/playlist?list=PLlrxD0HtieHi0mwteKBOfEeOYf0LJU4O1?WT.mc_id=academic-105485-koreyst) playlist is downloaded.
2. Using [OpenAI Functions](https://learn.microsoft.com/azure/ai-services/openai/how-to/function-calling?WT.mc_id=academic-105485-koreyst), an attempt is made to extract the speaker name from the first 3 minutes of the YouTube transcript. The speaker name for each video is stored in the Embedding Index named `embedding_index_3m.json`.
3. The transcript text is then chunked into **3 minute text segments**. The segment includes about 20 words overlapping from the next segment to ensure that the Embedding for the segment is not cut off and to provide better search context.
4. Each text segment is then passed to the OpenAI Chat API to summarize the text into 60 words. The summary is also stored in the Embedding Index `embedding_index_3m.json`.
5. Finally, the segment text is passed to the OpenAI Embedding API. The Embedding API returns a vector of 1536 numbers that represent the semantic meaning of the segment. The segment along with the OpenAI Embedding vector is stored in an Embedding Index `embedding_index_3m.json`.

### Vector Databases

For lesson simplicity, the Embedding Index is stored in a JSON file named `embedding_index_3m.json` and loaded into a Pandas Dataframe. However, in production, the Embedding Index would be stored in a vector database such as [Azure Cognitive Search](https://learn.microsoft.com/training/modules/improve-search-results-vector-search?WT.mc_id=academic-105485-koreyst), [Redis](https://cookbook.openai.com/examples/vector_databases/redis/readme?WT.mc_id=academic-105485-koreyst), [Pinecone](https://cookbook.openai.com/examples/vector_databases/pinecone/readme?WT.mc_id=academic-105485-koreyst), [Weaviate](https://cookbook.openai.com/examples/vector_databases/weaviate/readme?WT.mc_id=academic-105485-koreyst), to name but a few.

## Understanding cosine similarity

We've learned about text embeddings, the next step is to learn how to use text embeddings to search for data and in particular find the most similar embeddings to a given query using cosine similarity.

### What is cosine similarity?

Cosine similarity is a measure of similarity between two vectors, you'll also hear this referred to as `nearest neighbor search`. To perform a cosine similarity search you need to _vectorize_ for _query_ text using the OpenAI Embedding API. Then calculate the _cosine similarity_ between the query vector and each vector in the Embedding Index. Remember, the Embedding Index has a vector for each YouTube transcript text segment. Finally, sort the results by cosine similarity and the text segments with the highest cosine similarity are the most similar to the query.

From a mathematic perspective, cosine similarity measures the cosine of the angle between two vectors projected in a multidimensional space. This measurement is beneficial, because if two documents are far apart by Euclidean distance because of size, they could still have a smaller angle between them and therefore higher cosine similarity. For more information about cosine similarity equations, see [Cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity?WT.mc_id=academic-105485-koreyst).

## Building your first search application

Next, we're going to learn how to build a search application using Embeddings. The search application will allow students to search for a video by typing a question. The search application will return a list of videos that are relevant to the question. The search application will also return a link to the place in the video where the answer to the question is located.

This solution was built and tested on Windows 11, macOS, and Ubuntu 22.04 using Python 3.10 or later. You can download Python from [python.org](https://www.python.org/downloads/?WT.mc_id=academic-105485-koreyst).

## Assignment - building a search application, to enable students

We introduced our startup at the beginning of this lesson. Now it's time to enable the students to build a search application for their assessments.

In this assignment, you will create the Azure OpenAI Services that will be used to build the search application. You will create the following Azure OpenAI Services. You'll need an Azure subscription to complete this assignment.

### Start the Azure Cloud Shell

1. Sign in to the [Azure portal](https://portal.azure.com/?WT.mc_id=academic-105485-koreyst).
2. Select the Cloud Shell icon in the upper-right corner of the Azure portal.
3. Select **Bash** for the environment type.

#### Create a resource group

> For these instructions, we're using the resource group named "semantic-video-search" in East US.
> You can change the name of the resource group, but when changing the location for the resources,
> check the [model availability table](https://aka.ms/oai/models?WT.mc_id=academic-105485-koreyst).

```shell
az group create --name semantic-video-search --location eastus
```

#### Create an Azure OpenAI Service resource

From the Azure Cloud Shell, run the following command to create an Azure OpenAI Service resource.

```shell
az cognitiveservices account create --name semantic-video-openai --resource-group semantic-video-search \
    --location eastus --kind OpenAI --sku s0
```

#### Get the endpoint and keys for usage in this application

From the Azure Cloud Shell, run the following commands to get the endpoint and keys for the Azure OpenAI Service resource.

```shell
az cognitiveservices account show --name semantic-video-openai \
   --resource-group  semantic-video-search | jq -r .properties.endpoint
az cognitiveservices account keys list --name semantic-video-openai \
   --resource-group semantic-video-search | jq -r .key1
```

#### Deploy the OpenAI Embedding model

From the Azure Cloud Shell, run the following command to deploy the OpenAI Embedding model.

```shell
az cognitiveservices account deployment create \
    --name semantic-video-openai \
    --resource-group  semantic-video-search \
    --deployment-name text-embedding-ada-002 \
    --model-name text-embedding-ada-002 \
    --model-version "2"  \
    --model-format OpenAI \
    --scale-settings-scale-type "Standard"
```

## Solution

Open the [solution notebook](./solution.ipynb?WT.mc_id=academic-105485-koreyst) in GitHub Codespaces and follow the instructions in the Jupyter Notebook.

When you run the notebook, you'll be prompted to enter a query. The input box will look like this:

![Input box for the user to input a query](./images/notebook-search.png?WT.mc_id=academic-105485-koreyst)

## Great Work! Continue Your Learning 

After completing this lesson, check out our [Generative AI Learning collection](https://aka.ms/genai-collection?WT.mc_id=academic-105485-koreyst) to continue leveling up your Generative AI knowledge!

Head over to Lesson 9 where we will look at how to [build image generation applications](../09-building-image-applications/README.md?WT.mc_id=academic-105485-koreyst)!
