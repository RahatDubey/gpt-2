# ReadMe
I wanted to create an empathetic chatbot that can make you feel better if you are feeling sad. 

People most certainly like to be complimented. On reddit.com/r/toastme, you can see countless individuals receiving happiness from the compliments of complete strangers. They post a picture of themselves and a description of why they are feeling sad, and then their fellow redditors arrive in the comments to cheer them up. 

It seems to be a good system, but the problem is that you have to post a picture of yourself to the internet and then its there forever. This flattering chatbot that I am building would hopefully be able to be an app that you can download ( with more features, such as facial recognitions) that would be an AI companion. It would be able to remember conversations and help you feel better, without you having your saddest, most vulnerable moments documented on the internet forever.

I used OPENAI’s gpt2 implementation to create this . Its a transformer based model with hundreds of millions of parameters. Its conveniently pretrained and can be fine tuned to generate all types of text based on the input you give it. I spent many hours reading articles about this and reading the source code to understand how it works, how to use it, and how to explain this relatively novel deep learning concept. 

It would make more sense to now use  and fine tune Microsoft’s DialoGPT for conversation, and I plan to switch over to that, but a chatbot can be made out of this too without too much trouble.

# Data

The data I used to fine tune is publicly hosted on google’s big query dump of all reddit comments. I got all the toastme posts and comments and put them into a data frame on AWS SageMaker. I then fine tuned and trained GPT2 to predict

This is the SQL query I used on BigQuery to get the data

SELECT posts.title, comments.body ,comments.score
FROM `fh-bigquery.reddit_comments.2019_*` AS comments
JOIN `fh-bigquery.reddit_posts.2019_*`  AS posts
ON posts.id = SUBSTR(comments.link_id, 4) 
WHERE posts.subreddit = 'toastme' AND comments.score >= 1 AND posts.author != comments.author 
AND comments.body != '[removed]' AND comments.body != '[deleted]' AND parent_id = link_id AND NOT REGEXP_CONTAINS(comments.body, r"\[[a-zA-Z0-9-]\]") 

I had to make sure none of the comments were deleted or removed by moderators. I had to also make sure I didn’t capture discussion drifting off within threads so I only got the parent replies of each specific post. Parent replies are just about always  directed at OP to make them feel better, otherwise they are downvoted, and I made sure I only got comments with some upvotes. I also only used 2019 for a couple of different reasons. It is plenty of data ( more than a few megabytes of data is adequate to fine-tune GPT-2) and the subreddit felt like it had found a more consistent voice/ format by 2019 , though still inconsistent. Also, the time based references would at least be a little bit more relevant to the current day e.g: “No one wants to be my friend because I admitted I like Trump as a candidate” or “2017 is going to be your year!” In the replies.

I then saved that as a custom table ( BigQuery can’t save it as a csv or json because it was too big) and  connected to BigQuery on Amazon SageMaker (for more RAM and GPU power). Then I  selected everything from that custom table, and put it into a pandas data frame, that I could then convert into a text file in the proper format to be encoded and then trained on the model .

# Modeling: 

For fine-tuning a model, common practice is to make the learning rate 10 times smaller than the one used for scratch training. This makes sense because we don’t want to distort the original model too heavily, if its not too different from what we want it to look like now. In this case, both the original dataset and this one is comprised of English so I decided to conform to this standard, and it worked out okay. I also kept the batch size as 1 it seemed to have worked as a default for good results, and seemed to fit the model just the right amount. 

The loss went down steadily, which you can see in the output as it trains, meaning the model is getting better at predicting what is supposed to come next. It starts around 3.5-5 depending on the size of the data and the size of the model. It can get reasonably close to 0 if the data is large, and has straightforward patterns, (e.g “I lost my job” “Don’t worry you can definitely find another job!”) The hyper parameters are tuned to let it learn at the appropriate pace. For the demo model, I stopped it at 775 iterations, where the loss was 2.34, and the output was halfway decent:

You:”I’m a teen feeling insecure and lonely. Ive been self harming to ease the emotional pain im feeling." 

Friend:”You look kind and sweet! You look like a very approachable person who is looking for some positivity to help me feel a bit better about my looks. “

You: “21M, recently diagnosed bipolar. Kinda scared of what is coming up for me. Need some encouragement :(“

Friend: "#I dont need to tell you that you’re so hot lol"

You said: "13m I feel sad I am ugly"

Friend said: "You are hardworking, Chris..."

It has a hard time understanding context at the current moment for a couple of different reasons. This demo version wasn’t trained on enough data and training time. However, there is also a flaw with the data itself. Often times the users in the comments responding to the OP’s woes don’t even respond to what they wrote. They will go on tangents about the OP’s appearance, or sometimes give them generic encouragement. These kinds of things can make the chatbot itself seem impersonal. It is assured that more data definitely helps somewhat, however, because the human generated responses are still more relevant than the chatbot’s, and more data (and model parameters) .

The default temperature is 1, and that produces sentences that are mostly fine. Temperatures of over one produce garbled random nonsense ( more and more so exponentially), so it made no sense to move it in that direction.

I modified the temperature output slightly downward to .95, because I don’t care if the output is slightly more repetitive but still makes sense. A lower temperature can often create sentences that make more sense but less variety. This is fine for my use case. If the bot tells you are beautiful twice, that’s okay, as long as it didn’t go off topic or make up words. 

I used the “117m” model for proof of concept purposes temporarily, because its smaller , and more easily uploadable to GitHub and local machines. Its actually the 124 million parameter model if you look at the open AI source code, but the code here seems to be mistaken. The 335 million parameter definitely works better ( It got the loss down to 1.96 in less iterations) , and especially with more r/toastme data, but the 124 is good enough for demonstration purposes. To change it to 345m model, you need to change the default to 345m in training.py and encode.py ( not to be confused with encoder.py, which is just used for encoding the .txt file into a format that’s easier to train).


The steps to fine-tuning are written about in detail in Neil Sheppard’s blog, which you can find in the credits sections. The test data and full 2019 toastme2.text (the biggest dataset) file are in encoded npz format in the files folder ready to be re-finetuned (if desired) on the GPT-2 model. I also uploaded Jupyter notebooks that show the process of creating and using the demo model, and the command line actions are taken there as well. Those can be found in the Notebooks folder, specifically Chatbotter2.ipynb.

The chatter.py file can be ran as an executable once the model has been trained.  Within the file, you will find documentation on how to use it. I modified the code only slightly to be more convenient for my purposes but major credits go to Dlinne Bosman (GitHub in the credits). I put the data files I created in the src folder.

All of the other files are open source or from Nsheppard’s fork.




CITATIONS:
Chatbot https://blog.s-m.ac/GPT-2-ChatBot/
Dlinne Bosman (https://github.com/dwjbosman?tab=repositories)
https://medium.com/analytics-vidhya/understanding-the-gpt-2-source-code-part-1-4481328ee10b
http://jalammar.github.io/illustrated-gpt2/


# Forked from nsheppard:

Big thank you goes out to Neil Sheppard for slightly modifying the gpt-2 repo to incorporate finetuning.

Reference:  ["Beginner’s Guide to Retrain GPT-2 (117M) to Generate Custom Text Content"](https://medium.com/@ngwaifoong92/beginners-guide-to-retrain-gpt-2-117m-to-generate-custom-text-content-8bb5363d8b7f)

# gpt-2

Code from the paper ["Language Models are Unsupervised Multitask Learners"](https://d4mucfpksywv.cloudfront.net/better-language-models/language-models.pdf).

See more details in our [blog post](https://blog.openai.com/better-language-models/).


## Citation

Please use the following bibtex entry:
```
@article{radford2019language,
  title={Language Models are Unsupervised Multitask Learners},
  author={Radford, Alec and Wu, Jeff and Child, Rewon and Luan, David and Amodei, Dario and Sutskever, Ilya},
  year={2019}
}
```

## Future work

We may release code for evaluating the models on various benchmarks.

We are still considering release of the larger models.

## License

[MIT](./LICENSE)
