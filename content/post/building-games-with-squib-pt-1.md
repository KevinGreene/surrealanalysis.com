+++
date = "2016-06-30T01:14:12-04:00"
draft = false
title = "Building Games with Squib"

+++

As board games rise in popularity, many new groups focused on creating board games are popping up. Luckily, instead of expensive tools like InDesign, or Windows-centric tools like nanDeck, there are a number of alternatives. One actively developed alternative is [Squib](http://squib.rocks), a Ruby DSL, and we're going to be building a game with that.

To start, you need to have Ruby and Bundler installed. If you don't have either installed, you can install Ruby by following the directions on the [website](https://www.ruby-lang.org/en/downloads/). After Ruby is installed, run `gem install bundler` from the command line, and we're good to go! 

Now that we have the prerequisites installed, we need a game to build. We're going to start simple by building a custom themed triange deck, sometimes known as a [Pairs](http://cheapass.com/node/142) deck based on the game by James Ernest and Paul Peterson. To understand what types of games work well with this deck, I'd recommend reading [The Pairs Companion PDF](http://cheapass.com/sites/default/files/PairsCompanionBook.Scaffolding)

## Scaffolding

Squib has an excellent built in scaffolding tool. Do use this, you would first run `gem install squib`, followed by `squib init triangle-deck`. This would create a directory with all of the files needed to start. But, I find that generators can be confusing when learning a new tool, so we're going to build up all the files manually. 

To start, let's make a directory for our game. Inside that directory, we're going to make a number of files. 

We're going to be using Squib to create the files, and Guard to automatically generate them everytime there is a change, so paste the following in a new file named `Gemfile`:

{{< highlight ruby "linenos=inline" >}}
source 'https://rubygems.org'

gem 'squib'
gem 'guard-rake'
{{< /highlight >}}

Now that we have the libraries the project will use set up, we need some cards. To represent the data inside the cards, make a file named `cards.csv` in the same directory. 

Triangle Deck cards aren't very complex, so we only need one field - number, and fifty five rows. However a triangle deck has many duplicates, and we shouldn't need to type them all out. Luckily Squib includes a `Qty` feature. Instead of typing out fifty-five rows, we can type just nine if we add a `Qty` to each row. Copy the following into cards.csv:

```
number,Qty
"1",1
"2",2
"3",3
"4",4
"5",5
"6",6
"7",7
"8",8
"9",9
```

To Squib, this represents a deck with one "1" card, two "2" cards, and so on. Exactly what we want.

Now, we need to tell Squib how to interpret this data. To do so, we make a `cards.rb` file, and paste the following:


{{< highlight ruby "linenos=inline" >}}
require 'squib'

cards = Squib.csv file: 'cards.csv'

Squib::Deck.new(cards: cards['number'].size) do
  text str: cards['number'] 
  save format: :png
end
{{< /highlight >}}

In this file, we: 

* Load the Squib library
* Import the `cards.csv` file
* Create a new Squib deck as big as our cards CSV
* Display the value of "number" as a text element
* Output them to a PDF. 


Finally, we need a Rake file. This is the file we'll use to actually load our `cards.rb` file. Create a file called `Rakefile` and paste the following into it:

{{< highlight ruby "linenos=inline" >}}
require 'squib'

task default: [:cards]

task :cards do
  load 'cards.rb'
end
{{< /highlight >}}

We're ready to make a deck! Run `bundler exec rake`, and the folder `_output` should now exist. In it, we have all 55 of our cards in a PDF titled `output.pdf`!

We have a very basic deck, but it's very boring. Now it's time to add some visuals.

Part 2 coming soon
