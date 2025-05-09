# Geopolitics-for-Business---Group-Project

## LLM-Assisted Subindex – In Depth

This section reflects the actual methodology used in the current CGRI prototype, focusing on the experimental application of large language models to extract risk-related themes from company reports.

### Data
The documents we used for this part are public document from major companies, amon the ones we used are: annual reports, 10-k SEC filings, sustainability reports, proxy reports and others.
Raw data was sourced from PDF documents and extracted using PyPDF2, a Python library that allows for programmatic access to the text content within PDFs. Since large language models (LLMs) require plain text inputs for optimal processing.

We deliberately chose to minimize additional preprocessing (such as stemming, stopword removal, or heavy cleaning) to preserve the full contextual richness of the documents. LLMs like  3 can interpret nuanced language, and overprocessing could strip away key signals relevant for risk detection. Only basic formatting adjustments (e.g., line break handling and removal of extraneous characters) were applied to ensure readability without distorting meaning.

### The Model
The Transformer model used was Llama 3 – 70B, which was deployed for text generation and semantic similarity tasks via a Groq API. 
Groq is an  AI inference platform that allows execution of large language models, in this case in particular it allowed us to use a 70 Billion parameter model which would have been impossible with our machines; we also tried with other smaller model locally (such as Llama 3 – 8B) but the results were worse compared to the bigger model. 

Ideally we would have utilized an Instruct model, which is trained to just perform tasks, but instead Groq offered a chat optimized model, meaning it would respond in a chat like format, in simple terms it means that the model does not simply output the desired output, but usually pre-phrases it with “Here is your answer…” or analog statements, it also means that the model gives varied responded as in a chat-envirnoment, not giving always the same response is seen as a positive trait.
Nontherless, we made do with what we had, the only hiccup was that the output due to its nature needed manual processing to extract the relevant information.
Also another challenge we faced was the limitations on the API, which allowed for a maximum of roughly 30 thousand tokens per minute, meaning that we had do split our documents, we decided to have the model process the document in chunks and on the last iteration we would give back to the model its previous outputs so that it could give us a final response that was coherent for the whole document. 

We also performed a second analysis on the same documents, this however had more of a support role for the other indexes. We decided to perform Named Entity Recognition on these documents to find how many times each country is mentioned in the reports, this was meant as a sanity check for all other indexes. In particular we wanted a measure to compare with the LLM output to ensure that the LLM was not hallucinating but rather working correctly and coherently. This was thought from at the start as a problem that frequently arises when feeding huge documents (in our case some documents were as big as 600 pages) LLMs might go off rail and start generating output from their memories.
Named Entity Recognition (NER) is a natural language processing technique usually used to locate and classify entities, such as country names, organizations, and key individuals, within text. For this project, spaCy was used to extract named entities from the raw text, focusing specifically on geopolitical actors (e.g., countries, international institutions).
This added an interpretability layer to our workflow, helping ensure that outputs aligned with known entities and context.

When working with LLM, especially chat optimized ones, its extremely important to get the prompts right (also as re-doing the computation over and over would be costly as the Groq API works on a pay-per-token basis).
In order to do this we experimented with controlled prompt engineering, after a lot of trial and error we found a series of prompts that worked quite well. In the actual pipeline we used multiple prompts as we were processing the documents one piece at the time and we requred different prompts at different times, but the basic structure looked something like this:
system_message = """
You are an expert in geopolitical risk analysis. Given a text extracted from a company document, give insights and return a table with three columns:
1. Key geopolitical topics identified in the document.
2. A geopolitical risk score from 0 (low risk) to 10 (high risk).
3. A short justification for each score.

In the end, give a final score from 0 to 10 for the company based on the geopolitical risks identified.

Format the output as a markdown-style table.
Your work will be evaluated: if poorly done, you will be fired!
"""
Note:
This deliberately “high stakes” framing ("you will be fired"), while it may seem silly, improved output precision and structure, acting as an implicit signal to the model to avoid ambiguity. It also encouraged consistent formatting, useful for semi-automated downstream processing. Such tuning techniques, while informal, proved valuable for qualitative validation.



### Limitations
During this project we faced many challenges, and we overcame a good number of them, but still there are some we could not surpass, the main limitations we find out project to have are the following:
1.	A fully automated end-to-end pipeline has not yet been implemented; several steps currently require manual intervention.
2.	There is no scoring or confidence mechanism in place to quantify the relevance or severity of detected risks.
3.	The results remain qualitative and anecdotal, serving primarily as a cross-validation tool for existing geopolitical risk indexes rather than a standalone indicator. Further work is needed to establish statistical robustness and generalizability, which needs an investment in the acquisition of annotated data, in order to train specifically a model.

### Next Steps
This was a great project to tackle, we think that our prototype paves the way for deeper LLM integration into the Geopolitical Risk Index ecosystem, we feel we created a good proof of concept, but there are still many improvements to be made before it may be used in the real world. Some improvements we would like to see in the future are the following:
1.	Creating a labeled training set of geopolitical text segments: allowing for a sort of “geopolitical sentiment analysis” where instead of optimizing for text sentiment we optimize for geopolitical risk exposure
2.	Designing custom prompts for sector-specific geopolitical themes
3.	Incorporating outputs as flags into the CGRI scoring engine

Moreover, a promising direction for expanding this research would involve enhancing the use of the Named Entity Recognition. As we analyze the documents, we could leverage NER to systematically extract all country names mentioned within the text. This list of countries would then be used to inform and guide subsequent queries to the Large Language Model (LLM). By prompting the LLM with context-rich inputs that include each identified country, we could assess the sentiment or tone associated with that particular country in the context of the company’s activities.
This approach would enable a more detailed analysis of the relationships between the company of interest and the various countries it engages with, whether through business operations, partnerships, regulatory matters, or geopolitical exposure. By mapping sentiment to specific countries, we could generate an enriched, multidimensional index that not only captures general textual sentiment but also reveals how sentiment varies across geographies. In turn, this would allow for a meaningful comparison between the new LLM-driven sentiment index and the original country-level Geopolitical Risk Index.
Such an expansion would bridge the gap between advanced language model capabilities and classical geographical segmentation, offering a more dynamic and informative perspective on the global footprint and reputational landscape of the company under study.