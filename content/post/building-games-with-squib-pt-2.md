+++
date = "2016-07-02T01:14:12-04:00"
draft = false
title = "Building Games with Squib: Part 2, Design"

+++

In [the last post](/post/building-games-with-squib-pt-1), we covered how to set up a [Squib](http://squib.rocks) project from Scratch. In this post, we'll be enhancing our triangle deck with images and design.

<!--more-->

We have a basic triangular deck, but it has very little visual appeal. To add some visualization we are going to create a YAML file. Create a file called `layout.yml` in the directory, and add the following:

``` yaml
cut:
  x: 37.5
  y: 37.5
  width: 750
  height: 1050
  
number:
  x: 90
  y: 90
  width: 635
  height: 50
  align: center
  font: "Arial 40"
```

This layout adds a cut line for easily making the cards, and centers the number while making it a bit bigger. Both of these options are present in the YAML file created when running `squib new game`, but `title` has been renamed to `number`.

Our `cards.rb` file needs to know about this new layout. We also need to make sure that it's drawing the appropriate rectangle for `cut`, and that the number is being drawn with the `number` layout. Modify `cards.rb` to look like this:

{{< highlight ruby "linenos=inline" >}}
require 'squib'

cards = Squib.csv file: 'cards.csv'

Squib::Deck.new(cards: cards['number'].size, layout: "layout.yml") do
  rect layout: 'cut'
  text str: cards['number'], layout: "number"
  save format: :pdf
end
{{< /highlight >}}

When you recreate the PDF with `bundler exec rake`, we see that our cards are kind of far apart. This makes printing the game and cutting out cards kind of a hassle, so when we add our cards to the PDF, we need to trim the card to the cutline. Modify line 8 to be:

``` ruby
  save format: :pdf, trim: 37.5
```

After rerunning `bundler exec rake`, we see the cards are now being generated with easy cut lines. We are now at the point where we could easily print out this deck, cut it up, and play with others. But, we want to make it far prettier before we're at that point.

Before we improve our design, we should improve our workflow. Right now, everytime we make a change to any file, we are unnecessarily switching to a command line to run `bundle exec rake`. Instead, we're going to use a Ruby gem called [Guard]INSERTLINKHERE to automatically rerun the rake task whenever we change.

Create a Guardfile in the same directory, and copy the following into it:

``` ruby
guard 'rake', :task => 'cards' do
  watch(%r{^cards.rb})
  watch(%r{^layout.yml})
  watch(%r{^cards.csv})
end
```

Now, in the command line, run `bundle exec guard`. This will start a process that watches the CSV, Ruby code, or layout YAML files, and reruns the Rake task whenever it detects a change. We can confidently save changes to these files and have a new PDF created behind the scenes.

On the design side, while the `cut` layout is great for Print and Play, eventually we'd like to print these as Poker cards. To do this, we're going to add a `safe` block. This corresponds to The Game Crafter's recommendations, and is also present in the default Squib layout.

``` yaml
safe:
  x: 75
  y: 75
  width: 675
  height: 975
  radius: 16
  dash: 3 3
```

Instead of having the number in the center of the card, we're going to have it two places - the upper left and lower right. Replace the `number` section in `layout.yml` with the following:

``` yaml
number:
  width: 90
  height: 90
  font: "Arial 80"
  align: center
number_ul:
  extends: number
  x: 90
  y: 90
  valign: top
number_lr:
  extends: number
  x: 637.5
  y: 937.5
  valign: bottom
```

Because we want to share things like font size and alignment between all numbers, we modify `number` to include only these elements. Then, the children, `number_ul` and `number_lr`, extend `number` so by default they will have the same font, height, width, and alignment.

We then need to modify the `cards.rb` file to draw the rectangle, and draw two numbers instead of one. After the modifications it should look like this:


{{< highlight ruby "linenos=inline" >}}
require 'squib'

cards = Squib.csv file: 'cards.csv'

Squib::Deck.new(cards: cards['number'].size, layout: "layout.yml") do
  rect layout: 'cut'
  rect layout: 'safe'
  text str: cards['number'], layout: "number_ul"
  text str: cards['number'], layout: "number_lr"
  save format: :pdf, trim: 37.5
end
{{< /highlight >}}

After viewing the PDF, we can see that we now have relatively standard playing cards. The main thing left to add is an image. 

For triangle decks, I find it helps to pick a theme. In this particular case, we're going to make each card based off a particular south park character.

To start, we should make a folder called `assets`. Then, put the nine images you would like to use in this directory. In our case, I'm using the standard PNGs of Scott Malkinson, Craig Tucker, Tweek Tweak, Token Black, Butters Scotch, Kenny McCormick, Kyle Broflovski, Stan Marsh, and Eric Cartman, all of which were obtained from [the South Park Wikia](http://southpark.wikia.com/). The pictures will be saved based on their first name, e.g. the picture Eric Cartman should be `assets/eric.png`. It should be noted there might be some more prominent characters, but it's good to choose high resolution images, and those were not easily accessible for all desired characters.

In our `cards.csv` file, let's add a column between `number` and `Qty` called `character`. This will have the first name of the character we wish to show on a given card.

After updating the CSV file, it should look like this:

```
number,character,Qty
"1","eric",1
"2","kenny",2
"3","kyle",3
"4","stan",4
"5","tweek",5
"6","butters",6
"7","scott",7
"8","token",8
"9","craig",9
```

We should add a section to the layout to represent where this character's image will appear. Add the following to `layout.yml`:

``` yaml
character:
  x: 150
  y: 150
  width: :scale
  height: 700
```

Now, we need to add the PNG to each card. In the new Deck block, add

``` ruby
  png file: cards['character'].map { |c|  "assets/#{c}.png"}, layout: "character"
```
Most of our cards look great, with the notable exception of Eric Cartman, who is a bit too fat to fit on a card. Rather than try and crop him, or make him shorter, we're going to embrace it and center him. At the same time, a few other characters seem a little off center, as a peril of using non-standard size images.

We need to add custom layouts for each character in the YAML file, taking advantage of `extends`, and of additive x values. If we have a layout that extends another layout which defines an `x` position, we can set the child `x` to something like `x: += 30`, if we want the image moved 30 pixels to the right of the base image.

After typing all that out, your `layout.yml` file should have this for characters:

``` yaml
character:
  x: 150
  y: 150
  width: :scale
  height: 700
eric:
  extends: character
  x: += -125
kenny:
  extends: character
kyle:
  extends: character
kyle:
  extends: character
stan:
  extends: character
  x: += 30
tweek:
  extends: character
  x: += -30
butters:
  extends: character
  x: += 30
scott:
  extends: character
token:
  extends: character
craig:
  extends: character
  x: += 30
```

To make sure this is used properly, change line 10 of our `cards.rb` file to be

``` ruby
png file: cards['character'].map { |c|  "assets/#{c}.png"}, layout: cards['character']
```

The characters are a little bland on a white background, so let's change that to light grey by adding another line to the deck construction:

``` ruby
  background color: :light_grey
```

If you prefer, you could use a Hex string, or any other color found [here](https://github.com/rcairo/rcairo/blob/master/lib/cairo/colors.rb)

Now that we have the layout finalized, let's remove the `rect layout: 'safe'` line, as we'd prefer to avoid that when printing these out.

Our current state is available here: 


Stay tuned for Part 3, where we discuss how to go about getting these cards professionally printed. 
