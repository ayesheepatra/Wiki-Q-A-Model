# Wikipedia Question Answering System Report

## INTRODUCTION
This report presents the development of a Wikipedia Question Answering System, a tool designed to simplify the process of finding answers from Wikipedia articles. This system combines modern UI design principles with advanced NLP models to create an interactive and user-friendly interface.

## METHODOLOGY
The project is implemented in the below flow:
User Interface:
The user interface (UI) was developed using Streamlit, an open-source app framework for Machine Learning and Data Science projects. The UI allows users to input their queries and select topics for a tailored question answering experience. A sample screenshot of the UI is provided to demonstrate its functionality and layout.
 
### Working
The name of the chat bot is “Wisdom-Whisperer”. And the rightmost part of it is for the QnA between the User and the bot. The leftmost part or the sidebar is for selecting the topic. As of now it only allows user to select a particular topic and that topic is sent to the SOLR and it searches the top document on the specific topic selected by the user and retrieves the summary out of it.

### ChatterBot
The code initializes the "wiki-bot" chatbot using the ChatterBot library. It employs an SQLite database ('conversation_database.db') as a storage adapter to manage conversation data. The primary logic adapter is 'BestMatch' with a default response. The bot is trained using the 'chitchat_dataset,' a dataset containing predefined conversations. This ensures the chatbot's ability to provide contextually relevant responses. The 'BestMatch' logic adapter employs a rule-based matching algorithm, contributing to the chatbot's conversational capabilities and overall functionality.

### Chat-End Classifier
We designed a classifier to distinguish between query which wants to continue chatting and a query which wants to terminate the chat. We have used a zero-shot classifier and “DeBERTa-v3-large-mnli-fever-anli-ling-wanli” transformer for text classification.
For this model we tried and used the following candidate levels that gave the highest accuracy: “continue conversation” and “end conversation”.If the classifier gives the results as “continue chat”, then the query is passed to wiki-chat classifier.

### Wikipedia vs. Chat Classifier
A zero-shot classifier utilizing the ` DeBERTa-v3-large-mnli-fever-anli-ling-wanli ` transformer was employed to distinguish between informational queries directed towards Wikipedia and conversational requests meant for chitchat. This model allows the system to understand the intent behind a user's question without the need for explicit training on those categories.
After trying multiple models, deBERTa-v3-large-mnli-fever-anli-ling-wanli gave the highest accuracy while distinguishing between the labels wiki information vs normal chitchat.
A sample result of the Wikipedia vs. Chat Classifier when the user inputs “hello, how are you?”:
 
### Topic Classifier
To accurately categorize queries into topics, the `DistilBERT_ZeroShot` model, known for its efficiency and effectiveness, was used. This step ensures that questions are directed to the relevant sections of Wikipedia for precise and accurate information retrieval. The retrieval of documents are done 
After trying multiple models like mobilebert-uncased-mnli and large models provided by the hugging face library, DistilBERT_ZeroShot gave the best accuracy and runtime while distinguishing between the labels wiki information vs normal chitchat
A sample result of the Topic Classifier when the user inputs “When is the next presidential elections?”:

## RAG Implementation
Once the query gets classified as wiki, the raw query is sent to SOLR for documents retrieval. SOLR produces all the documents that match the unique words in the query and sends the output in a json format ranked by their tf-idf scores. 
All the summaries are concatenated and and written into a word file that works as a text loader for the RAG model. 
We have langchain library for text classification and defined a prompt to tell the intelligent to understand the truthiness of the summaries and generate a intelligent summary of the same.

## Working of the classifiers
### Indexing and Document Retrieval:
The indexing of documents scraped from Wikipedia was managed using Solr, a powerful search platform. The system performs searches on two key fields of the scraped documents: the topic and title. Titles are subject to wildcard searches based on specific keywords from the user's query, ensuring the retrieval of the most relevant documents.
For example, if preprocessed user query looks something like this “know covid 19”. 
The search is conducted by constructing a Solr query that includes wildcard searches for the given terms in the document titles. The constructed query is then sent to the Solr instance, and the response is processed. The code does not explicitly implement a custom scoring mechanism like TF-IDF; instead, it relies on Solr's built-in scoring system, which calculates relevance scores for each document based on factors such as term frequency, inverse document frequency, and field length normalization. The search results, including document metadata like revision ID, title, topic, and summary, are then printed in a tabular format. While the code doesn't explicitly fetch the top N documents, Solr's default behavior is to return results sorted by relevance score, presenting the most relevant documents first.
 

## VISUALISATION
1.	Save N number of conversations for analytics and visualization.
We have saved the history of conversations and used it as a dataset to analyze 
Plot of bot responses and user input as a measure of diversity:
 
2. Understand which topics the chatbot is able talk more about and where most of the errors are
occurring. Also analyse the user queries and what type of queries are fetching more relevant
utterances.
The bot performs well with normal chit-chat and also it classifies well whether the user input is for chatting or farewell. Below is a bar-plot of the number of classifications performed correctly and incorrectly for continue chat vs bye and wiki information vs chat.
  
3. There are certain instances where the model does not distinguish well whether to continue chat or terminate the process. As the table shows, for a user-input of “so you are an alien”, bot generated classification is “bye” where the ground truth is to “continue chat”.
 
4. Based the previous point can we group certain queries? Do query reformulation increases the chat
coherency?
Query formulation will increase the chat coherency. If the model learns from the history of the current session, then it will understand better on what to reply next for the present query. 
For the wiki vs chat classifier, the model classifies the below query incorrectly: Here we see that, even for a personal chat, it classifies as a wiki query by considering the unique words in the query. If the model had learnt from the previous queries or if it had considered words like “you” as a personal keyword, then the model could have performed better in such cases.

6. Be creative and come up with your own ideas.
Implementing RAG for normal chit chat could have been a better performer.

# CONCLUSION
The Wikipedia Question Answering System represents a significant step towards making information retrieval more accessible and interactive. Through the use of cutting-edge NLP models and an intuitive UI, users can effortlessly navigate vast amounts of information to find the answers they need.

