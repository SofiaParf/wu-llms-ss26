Before all steps, I created a csv file, where I stored not only prompts, but also answers to them, which was provided by the class.

For the first model I used a library OpenAI to call on OpenAI API. To generate prompts I used following parameters:
- model = gpt-4.1-mini
- max_completion_tokens = 250 
- temperature = 0 
I chose the last two parameters as stated, as the task was to give succinct (max_completion_tokens, not too long) and precise (temperature, more deterministic output). 
For this model there was no pre-training, as this model is a proprietary model pre-trained by OpenAI and only inference took place at this point. 

For the fine-tuning model I chose tiiuae/falcon-rw-1b. Falcon-RW-1B is an open-source model developed by TII and pre-trained on the RefinedWeb dataset. It has high performance for its size and ideal for efficient deployment. As for the fine-tuning data, I created a dataset partially with the help of library OpenAI. I consulted a newer, more accurate model gpt-5 and constructed a question-answer datasets. After highlighting the legal topics ( for example, corporate tax law etc.) it created a 600-line dataset with legal references (§). 
My chosen method of fine-tuning is LoRA via the PEFT library.Each training example was formatted as: Frage … Antwort … 
Hyperparameters: 
Base model: Falcon-RW-1B (~1B parameters)
Fine-tuning method: LoRA
Rank (r): 4 - defines LoRA can adjust 4 parameters. Although this number is stable and requires low memory to avoid crashes, it has limited learning capacity
LoRA alpha: 16
Dropout: 0.05
Batch size: 1
Gradient accumulation steps: 8
Effective batch size: 8
Learning rate: 3e-4
Number of epochs: 1 - Unfortunately, i had to set the number of epochs to 1, as there was a high risk of crushing. This number is far from ideal, as the model only sees the dataset once and I had to consider the risk of underfitting 
Max sequence length: 128 tokens 

No retrieval-augmented generation (RAG) approach was used in this project.

I evaluated model performance using three metrics:
* BERTScore (F1): measures semantic similarity between generated and reference answers
* Legal Match Score: extracts legal references (§) and compares them using Jaccard similarity. As per task, there was an emphasis placed on the importance of the correct legal references, I decided to check on them.  However, the model could give a correct answer without mentioning the legal sources. For this reason I calculated the final score as follows:
Final Score = 0.7 × BERTScore + 0.3  × Law Score 

     Model  Avg Law Score  Avg BERTScore  Final Score
0  Model 1       0.245763       0.668986     0.542019
1  Model 2       0.068119       0.641632     0.469578

Based on the evaluation metrics, the first model performs best overall, achieving a higher final score (0.54 vs 0.47).
It produces more complete and semantically accurate answers, despite slightly lower emphasis on legal citations.
The reasons might be following: 
- Differences in the models: GPT-4.1 mini (approximately 7 billion to 8 billion parameters) vs Falcon-RW-1B (~1B parameters)
- When training the fine-tuning model, too little training data was input 
- The  unfavourable setting of the parameters for the LoRA 
- Max length of sequences: 250 vs 128 
The first model gave more open answers, whereas the answers in the second model simply got cut off, significantly affecting the performance. As models tended to put the sources at the end of the answer, it could be one of the reasons why the law score for the second model is so low. 

The main types of errors observed include:
* Shortened answers (Model 2): caused by shorter max sequence length (128 tokens)
* Missing legal references (§): especially when answers are cut off
* Inconsistent citation style: variation in how legal provisions are referenced

I encountered several limitations during the project, the main being computational constraints. Often Google Colab gave a warning that the amount of free tokens has been used up and I had to wait for them to restore before continuing. For this reason the debugging progress took a long time. 

