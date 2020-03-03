# ReadMe

I wanted to create an empathetic chatbot that can make you feel better if you are feeling sad. 

People most certainly like to be complimented. On reddit.com/r/toastme, you can see countless individuals receiving happiness from the compliments of complete strangers. They post a picture of themselves and a description of why they are feeling sad, and then their fellow redditors arrive in the comments to cheer them up. 

It seems to be a good system, but the problem is that you have to post a picture of yourself to the internet and then its there forever. This flattering chatbot that I am building would hopefully be able to be an app that you can download ( with more features, such as facial recognitions) that would be an AI companion. It would be able to remember conversations and help you out, without everyone in the world knowing.

I used OPENAI’s gpt2 implementation to create this . Its a transformer based model with hundreds of millions of parameters. Its conveniently pretrained and can be fine tuned to generate all types of text based on the input you give it. 

The data is publicly hosted on google’s big query dump of all reddit comments. I got all the toastme posts and comments and put them into a data frame on AWS SageMaker. I then fine tuned and trained GPT2 to predict

It would have made more sense to use  and fine tune Microsoft’s DialoGPT, but a chatbot can be made out of this 

CITATIONS:
Chatbot https://blog.s-m.ac/GPT-2-ChatBot/
Dlinne Bosman (https://github.com/dwjbosman?tab=repositories)
https://colab.research.google.com/drive/1VLG8e7YSEwypxU-noRNhsv5dW4NfTGce#scrollTo=-xInIZKaU104


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
