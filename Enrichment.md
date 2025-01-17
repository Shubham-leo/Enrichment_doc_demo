# Enrichment 
Enrichment task analyze news articles to extract key information. It identifies organizations and individuals mentioned, evaluates sentiment for sentences with specific keywords, generates concise summaries, detects and classifies events, and categorizes the news into predefined labels. Enriched data is then updated in a database for further use.
[Flow Chart](https://lucid.app/lucidchart/f79bba75-30f0-4281-bde0-8fb17ce4f99c/edit?viewport_loc=-345%2C-274%2C1752%2C814%2C0_0&invitationId=inv_0c5aa225-5669-48ef-b442-5cb6786e7e87)


1. **Initialize Parent Function `perform_enrichment `**
   - **Load Resources**: 
     - Load Natural Language Processing (NLP) models, sentiment analysis models, and tokenizers.
     - Set up summarization models (`T5-small` and fallback `DistilBART`).
     - Prepare the multi-label event classification and news classification models.
   - **Inputs Required**: 
     - `article` (text of the news article to process).
     - `tsa_keyword` (target keyword for filtering sentences).

2. **Execute Child Functions Sequentially**:
   - **Step 1: Named Entity Recognition (NER)**
     - **Function**: `extract_org_frequency`
       - Inputs: `article`, pre-loaded `nlp` model.
       - Process: Identify and count mentions of organizations.
       - Output: A dictionary of organizations with their frequencies.
     - **Function**: `extract_person_gliner`
       - Inputs: `article`, pre-loaded `nlp` model.
       - Process: Identify and count mentions of persons.
       - Output: A dictionary of persons with their frequencies.

   - **Step 2: Sentiment Analysis**
     - **Function 1**: `extract_sents`
       - Inputs: `article`, `tsa_keyword`.
       - Process: Extract all sentences from the article containing the `tsa_keyword`.
       - Output: A list of filtered sentences.
     - **Function 2**: `get_sentiment_modified`
       - Inputs: Filtered sentences, sentiment analysis model.
       - Process: Perform sentiment analysis on each sentence to calculate sentiment scores.
       - Output: Sentiment scores for individual sentences.
     - **Function 3**: `aggregate_sentiment_news_level`
       - Inputs: Sentence-level sentiment scores.
       - Process: Aggregate sentiment scores at the news level.
       - Output: A consolidated sentiment score for the entire article.

   - **Step 3: Summarization**
     - **Function 1**: `compute_summary_updated`
       - Inputs: `article`, `tsa_keyword`, pre-loaded `T5-small` summarization model.
       - Process: Generate a summary specific to the keyword.
       - Output: A keyword-based summary.
     - **Function 2**: `compute_summary`
       - Inputs: `article`, pre-loaded `DistilBART` summarization model.
       - Process: Generate a general summary as a fallback.
       - Output: A general article summary.

   - **Step 4: Event Detection and Classification**
     - **Function**: `multi_label_event_classification`
       - Inputs: `article`, event classification model and tokenizer.
       - Process: Detect possible events in the article and classify them into predefined event categories.
       - Output: Top 2 event categories.

   - **Step 5: News Multi-Class Classification**
     - **Function**: `get_news_labels`
       - Inputs: `article`, news classification model.
       - Process: Categorize the article into predefined news classes.
       - Output: Predicted news categories.

3. **Consolidate Outputs**:
   - Combine results from all child functions into a structured format:
     ```json
     {
       "organization_frequency": {...},
       "person_frequency": {...},
       "sentence_sentiment_scores": [...],
       "news_sentiment_score": ...,
       "keyword_based_summary": "...",
       "general_summary": "...",
       "event_categories": [...],
       "news_categories": [...]
     }
     ```
   - Add metadata (e.g., timestamp, source of the article, or any additional required fields).

4. **Error Handling**:
   - Use `try-except` blocks around each child function to catch and log errors.
   - For failed steps, return a placeholder or skip to the next step without halting the entire process.

5. **Return Final Output**:
   - Return the consolidated output.
   - Optionally, save the output to a database or upload it to a storage service (e.g., Azure Blob or AWS S3).
