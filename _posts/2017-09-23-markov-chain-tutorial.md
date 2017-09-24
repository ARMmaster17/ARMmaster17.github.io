---
layout: post
title:  "Writing a Markov-Chain Chatbot"
date:   2017-09-23 12:00:00 -0500
categories: markov chain ai chatbot jeffbot tutorial
---

A while back, I wrote [Jeffbot](https://github.com/ARMmaster17/JeffBot) as a fun side project because I was bored between classes in college. JeffBot is simply a chatbot with the intelligence of a two-year-old. It learns words and what words can/can't be used sequentially in a sentence. At first, it spews out pure garbage or just repeats back what you just said. However, over time, JeffBot learns how to make what appears to be intelligent english.

JeffBot itself is a complex app split over three different Heroku dynos and a particularly large database running on top of Rails/Sinatra/ASP.NET/I'm not even sure what anymore. In this article, I'm going to show you how to rebuild an N2 version of JeffBot in pure Ruby that can run locally with no external service/library dependencies.

First, you will need to set up a directory for JeffBot. Create a directory called `JeffBot` and `cd` into it. From now on, this directory will be referred to as `~/`. Inside this directory, create a file called `main.rb` and paste this code in.

{% highlight ruby linenos %}

# ~/main.rb

def learn(input, training_data)
    grams = input.split(' ').each_cons(2).to_a
    grams.each do |gram|
        entries = training_data.select { |entry| entry.w1.eql?(gram[0]) && entry.w2.eql?(gram[1]) }
        if entries.count == 0 || entries.nil?
            training_data << Struct::Ngram.new(gram[0], gram[1], 1)
        else
            entries.first.count += 1
        end
    end
end

def respond(input, training_data)
    word = input.split(" ").sample
    sentence = word
    for i in 0..10
        entries = training_data.select { |entry| entry.w1.eql?(word) }
        if entries.count == 0 || entries.nil?
            sentence << "."
            break
        else
            new_word = entries.sample.w2
            sentence << " " + new_word
            word = new_word
        end
    end
    return sentence
end

Struct.new("Ngram", :w1, :w2, :count)

learning_data = []

while true
    print ">"
    input = gets.chomp
    if input.eql?("exit")
        exit(0)
    end
    learn(input, learning_data)
    puts respond(input, learning_data)
end

{% endhighlight %}

Let's go through this code block by block to see what it does. Let's start in the same place that the interpreter starts, line 32. This defines a struct called an N-gram. This represents a relationship between how likely one word is to appear after another. Take a look at the following input string:

```
This is a sentence.
```

Our `learn()` function that we are going to cover in a minute is going to split that sentence into 2n N-grams like so.

```
[This, is]
[is, a]
[a, sentence]
```

Assuming this is the only training data provided, if we start with "a", our chatbot would choose "sentence" as the next word. If multiple relations exist for one word, a random selection is made based on the choices (utilizing the `count` attribute is left as an exercise for the reader).

The remainder of the code creates a `learning_data[]` array that we use to store training data. It also provides a loop to take input from the user, and exit the application if they have requested to do so by typing "exit" at the prompt. Towards the bottom of the code block, we see calls to `learn()` and `respond()`. These two methods form the backbone logic of our chatbot.

Now let's take a look at the `learn()` method. We call it with two parameters, `input` and `training_data[]`. The first thing we need to do is convert `input` into a series of 2N-grams like so.

{% highlight ruby linenos %}

grams = input.split(' ').each_cons(2).to_a

{% endhighlight %}

Next we iterate over the list of `grams`, with `gram` representing the relation we are currently looking at.

{% highlight ruby linenos %}

entries = training_data.select { |entry| entry.w1.eql?(gram[0]) && entry.w2.eql?(gram[1]) }

{% endhighlight %}

This line pulls out an N-gram relation if it already exists in our training data where both the first word and the second word match in the `gram` we are currently looking at.

{% highlight ruby linenos %}

if entries.count == 0 || entries.nil?
    training_data << Struct::Ngram.new(gram[0], gram[1], 1)
else
    entries.first.count += 1
end

{% endhighlight %}

This `if` block runs some logic on the results from the previous query of `training_data[]`. If a relation already exists, we increment the count. Otherwise we create a new `Ngram` Struct and shove it into `training_data[]`.

Now let's move on to the `respond()` method. We call this with the same parameters as `learn()`. Our first line picks the first word based on the words that the user wrote at the prompt. We use this word to start our `sentence` and to initialize `word` that we will utilize in a loop.

{% highlight ruby linenos %}

    word = input.split(" ").sample
    sentence = word

{% endhighlight %}

Next we enter a `for` loop. I set an arbitrary limit of 12 words in a sentence, you can increment this by increasing the range that the `for` loop iterates over.

{% highlight ruby linenos %}

entries = training_data.select { |entry| entry.w1.eql?(word) }

{% endhighlight %}

Within this `for` loop, we use similar logic from the `learn()` method. The difference is that we only have the first part of the N-gram (`w1`). If the number of matchine entries is zero, we add a period to the sentence and return it to the prompt loop.

{% highlight ruby linenos %}

if entries.count == 0 || entries.nil?
    sentence << "."
    break

{% endhighlight %}

If matches are found, we assign it to `new_word`. Then we add it to `sentence` and move it to `word` so we can use it as our new `w1` on the next loop iteration.

That's all there is to it. Run it by `cd`-ing into your project directory and running `ruby ./main.rb` from the terminal. When you recieve the `>` prompt, enter a string of words. Your new chatbot should reply back with some garbage. Talk with it some more, the more input you provide, the faster it can learn and respond with some more intelligent-sounding english.

To quit, just type "exit" at the `>` prompt. Note that learning data is stored in memory. This means that when you quit, all training data is lost. When you run the program again, you are starting with a fresh slate. For more permanent storage, take a look at the `pg` ruby gem paired with the PostgreSQL database service.