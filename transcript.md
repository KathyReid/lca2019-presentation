# INTRODUCTION


welcome in Maori



thank you to the volunteers, AV?

mention Duncan MacNeil coined the term - photo of him from LCA2018

# OVERVIEW

## Voice within the context of evolving user interfaces

Not so long ago - a couple of decades perhaps, speaking with a computer was relegated to the realms of science fiction. We saw cars that could talk;

IMAGE: KITT in Knight Rider

computers that could talk - "hello computer" _in Scottish accent_

IMAGE: Star Trek computer

and even holograms that could talk;

IMAGE: Time Trax

Just like many of the technology showcased in these series - Knight Rider, Star Trek, Time Trax, voice user interfaces are quickly moving from science fiction to science fact. Advances in processor speed, compute power, machine learning and neural networks are quickly making voice user interfaces a reality.

However, commercial, proprietary solutions have led the way here - Apple, with Siri, Amazon with Alexa, Microsoft with Cortana, Samsung with Bixy. There are many reasons for this - which I'll discuss throughout the presentation, but like many of you, I'd really like to see some open source solutions available which rival the maturity and feature completeness of the proprietary solutions.

## Introduction to the voice stack

So,

_How many people in the audience are familiar with some type of speech recognition or text to speech service? *count how many* OK, that's great!_

OK, this will probably be a recap for many of you, but for the benefit of those who are new to voice, I'm going to provide an introduction to the voice stack.

Just like there are **stacks** of software for things like web servers or databases, or application servers, voice stacks need certain layers as well. So, in a typical web application, let's take it from the back end to the front end, you might have a database layer - postgres or mysql or similar, you might have some sort of database caching layer - Redis or similar, you might have an application layer - PHP with Laravel or Python with Django - that sort of thing, then with a web server in front of that - nginx or apache, and probably a proxy in front of that.

So, a voice stack has a lot of components as well, and they're described in this diagram, by my Mycroft AI colleague, David Smehlik.

IMAGE: Anatomy of a voice interaction

In a voice interaction you begin with a **Wake Word** - also called a **Hot Word** - which is used to prepare the voice assistant to receive a command. Next, we use a **Speech to Text** engine to transcribe an **Utterance** from voice sounds into written language. Next, we use an **Intent parser** to determine what _type_ of command the user wanted to execute. Then, we select a command to run and execute it. Next, we turn written language back into voice sounds again using a **Text to Speech** engine.

Does that make sense? OK, great.

Now, I'm going to dive a little deeper into each of the different layers of the voice stack, and we're going to explore some of the challenges at each layer of the voice stack, using open source software examples.

# WAKE WORD

## PocketSphinx

One of the earliest Wake Word engines used was PocketSphinx. PocketSphinx is part of the broader CMU Sphinx project from Carnegie Mellon University. PocketSphinx recognises Wake Words based on something called _phonemes_. What's a _phoneme_? Good question!

## Phonemes

IMAGE: Phonemes

A phoneme is the smallest unit of sound that distinguishes one word from another in a particular language.

Different languages have different phonemes – for example, it’s hard to approximate the Indonesian trill “r” in English (this phoneme is the sound you make when you “roll” your r sound).  Using phonemes for Wake Word detection can also therefore be a challenge for people who are not native speakers of a language – as their pronunciation may differ from the “standard” pronunciation of a Wake Word.

The other challenge with using Phonemes in Wake Words is that certain Phonemes sound very similar to each other.

IMAGE: Similar sounding phonemes

This is one of the reasons why Wake Words for voice assistants are often very different-sounding to other words - that is, they're _differentiated_. Not much sounds like `Alexa` or `OK Google` or `Hey Mycroft`. This is a deliberate choice to make it easier to distinguish between phonemes.  If you're thinking about training your own Wake Word, then this is a factor you need to consider as well.

But phonemes are not the only way to handle Wake Words.

## Snowboy

Snowboy is another Hot Word detection engine – available under both commercial and open source licenses. Snowboy differs from PocketSphinx in that it doesn’t use phonemes for Wake Word detection; it instead uses a neural network that is trained on both false and true examples to differentiate between what is and isn’t a Wake Word.

## Precise

Mycroft AI’s Precise Wake Word engine works in a similar way – by training a recurrent neural network to differentiate between what is and isn’t a Wake Word. This is then trained on samples that *are* and *are not* Wake Words to improve the detection accuracy.

## Challenges with Wake Words

SLIDE: Wake Word challenges

### Always listening

Even though all of these Wake Word detectors work `on device` - meaning that they don't need to send data to the cloud, they are `always listening`. That means that if the device on which the Wake Word listener is connected to the internet, it is _possible_ that Wake Word recordings are sent over the cloud. So this is something that you need to be very aware of with a voice assistant - make yourself aware of how that information is being used. At Mycroft for example, our users must opt-in before recordings are transmitted back to our servers for training to improve accuracy. If the user hasn't opted in, then the recording is discarded.

We're actually seeing some interesting responses to this.

IMAGE: Project Alias

This is Project Alias, and this is like a "hat" for Alexa or Google Home, and what it does is run a small fan - it essentially "blocks" the device from hearing the Wake Word it's programmed with. Instead, you set a different Wake Word on the Project Alias device, and it acts essentially as a Wake Word proxy - and outputs the "real Wake Word" when you say the "proxy Wake Word" - it's really clever actually. Or, I mean, you could just use a voice assistant that doesn't sell your privacy to the highest bidder I suppose ;-)

### Accuracy

IMAGE: To be found

I touched on accuracy before, but it's worth expanding on here, as it really is a challenge, not just for open source Wake Word engines, but for all Wake Word engines. It's a key challenge for a number of reasons.

SLIDE: False positive vs false negative

A Wake Word engine has to be trained for these four states  _go through states_. The way we do this is to train it on both positive and negative samples - that is, recordings that are, and are not, the Wake Word. It sounds really simple, doesn't it? Yeah... nah!

What we've seen happen in reality at Mycroft AI - I can't speak for other voice platforms - is that there are lots of **biases** in the Wake Word samples;

* We find that there are significantly more samples of male-sounding voices than female - by a factor of at least ten to 1
* In our dataset, there are significantly more samples of American accents, as opposed to European, Asian or Latin American accents

Combined, what this means is that if you're a woman, who's not American, you're _much_ less likely to have the Wake Word engine correctly identify you. Really, the only way to combat these sorts of biases is to have a greater range of samples to train on. However, that also presents its own issues. At one point we considered filterng out a percentage of male Wake Word samples, to try and remove some of the bias in the data set - but this would mean tagging users and samples with a gender flag or some form of identifier - and as a privacy-focussed company, that wasn't something that we were comfortable doing - similar with accents which might indicate cultural or ethnic heritage.

So that's some of the challenges we have with Wake Words.
Let's move on to Speech to Text.


# SPEECH TO TEXT

SLIDE: Speech to Text

Accurate Speech to Text conversion is one of the most challenging parts of the open source voice stack.

Kaldi is one of the most popular Speech to Text engines available, and it has several “models” to choose from. In the world of Speech to Text, a “model” is a neural network that has been trained on specific data sets, using a specific algorithm. Kaldi has models for English, Chinese and some other languages too. One of Kaldi’s most attractive features is that it works “on-device”.

SLIDE: Common voice languages 

Mozilla’s DeepSpeech implementation – along with the related Common Voice data acquisition project – aims to also support a wider range of minority languages. As at the time of writing, the compute requirements for DeepSpeech mean that it can only be used as a cloud implementation – it is too “heavy” to run ‘on device’.


## STT Challenges


One of the biggest challenges with Speech to Text is training the model. Mycroft AI have partnered with Mozilla, enabling our community to help train DeepSpeech. We then pass the trained data back to Mozilla to help improve the accuracy of their models.












# SKILLS

## General challenges with Skills



## Fallback Skills
Confidence of which fallback is activating



## Voice user interaction

Haber's classification of contexts

Wake Word detection



Speech to Text

Accurate Speech to Text conversion is one of the most challenging parts of the open source voice stack.
Kaldi is one of the most popular Speech to Text engines available, and it has several “models” to choose from. In the world of Speech to Text, a “model” is a neural network that has been trained on specific data sets, using a specific algorithm. Kaldi has models for English, Chinese and some other languages too. One of Kaldi’s most attractive features is that it works “on-device”.
Mozilla’s DeepSpeech implementation – part of their broader Common Voice project – aims to also support a wider range of minority languages. As at the time of writing, the compute requirements for DeepSpeech mean that it can only be used as a cloud implementation – it is too “heavy” to run ‘on device’.
One of the biggest challenges with Speech to Text is training the model. Mycroft AI have partnered with Mozilla, enabling our community to help train DeepSpeech. We then pass the trained data back to Mozilla to help improve the accuracy of their models.
Intent Parsing

A strong voice stack also needs to ensure that the intent of the user is accurately captured. There are several open source intent parsers available.
Rasa is open source and widely used in both voice assistants and chatbots.
Mycroft AI uses two intent parsers. The first, Adapt, uses a keyword matching approach to determine a confidence score, then passes control to the Skill, or command, with the highest confidence. Padatious takes a different approach, where examples of entities are provided, so that it can learn to recognise an entity within an utterance.
One of the challenges with Intent Parsers is that of intent collisions – imagine the utterance;
“Play something by The Whitlams”
Depending on what commands or Skills are available, there may be more than one that can handle the intent. How does the Intent Parser determine which one to pass to? At Mycroft AI we have recently implement our Common Play Framework, which assigns different weights to different entities, leading to a more accurate overall intent confidence score.
Text to Speech
At the other end of the voice interaction lifecycle is Text to Speech. Again, there are several opensource TTS options available. In general, a TTS model is trained by gathering recordings of language speakers, using a structured corpus – or set of phrases. Machine learning techniques are then applied to synthesize the recordings into a general TTS model – usually for a specific language.
MaryTTS is one of the most popular, and supports several European languages.
Espeak has TTS models available for over 20 languages, although the quality of synthesis varies considerably between languages.
Mycroft AI’s Mimic TTS engine is based on CMU Flite and has two voices available for English. The newer Mimic 2 TTS engine is a Tacotron-based implementation, which is a less robotic, more natural sounding voice. Mimic runs on-device, while Mimic 2, due to compute requirements, currently runs in the cloud. Additionally, Mycroft AI have recently released the Mimic Recording Studio, an open source, Docker-based application that allows people to take recordings which can then be trained using Mimic 2 into an individual voice.
This solves one of the many problems with TTS – having natural-sounding voices available in a range of genders and languages / dialects.
Parting notes
As you can see, there is an emerging range of open source voice tools becoming available, each with their own benefits and drawbacks. One thing is for certain though – the impetus towards more mature open source voice solutions that protect privacy is here to stay!


image credits
