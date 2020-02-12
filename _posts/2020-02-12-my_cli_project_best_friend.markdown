---
layout: post
title:      "My CLI project Best Friend"
date:       2020-02-12 05:32:30 -0500
permalink:  my_cli_project_best_friend
---

<center><img src="https://i.imgur.com/sJQlX8R.png" alt="drawing" width="400"/></center>


This is one of the first, if not the first project I created from scratch, so I was nervous and excited at the same time.

I didn't know where to start, so I begin to find a web that was easy to 'scrap', and at the same time challenging, the winner was dogtime.com
a website where I can find info about dogs and more relevant for my idea breeds names and info.

so I started creating the skeleton using bundler. easy peasy.

```
bundle gem best_friend
```

So at least now I'm not facing empty files, step by step.
Next is to scrap all the info I need so I started to create my Scraper class.

There was a lot of info to collect. Ideas don't stop to come to my head.
I decided to scrap all breeds and breeds by group, so I created two methods to do so,but I needed the invaluable help the [nokogiri.](https://nokogiri.org/)

```
  def self.group_by(url)
     doc= Nokogiri::HTML(open(url))
    hash = {}
    doc.css('li.item.paws').collect do |div| 
      key = div.css('h3').text
      value = div.css('a').attribute('href').value
      hash[key.to_sym] = value
    end
    hash
  end

    def self.breeds_by_url(url)
      doc = Nokogiri::HTML(open(url))
      hash={}
      breeds_list = doc.css('.list-item-title').collect do |breed| 
        hash[breed.text.to_sym] = breed.attribute('href').value
      end
      hash
    end
```

Now I got a hash of breeds and its urls.
next scrap all that info and pass it to the class Breeds.

```
def self.breed_info(url)
    doc = Nokogiri::HTML(open(url))
    hash = {
      name:doc.css('h1').text,
      bio: doc.css('p').first.text,
      characteristics:[],
      stats: {}
    }
    # Collecting Vital Stats
    doc.css('div.vital-stat-box').collect do |stat|
      stats = stat.text.split(':')
      hash[:stats][stats[0]] = stats[1]
    end
    # Collecting Main Characteristic and Stars Ratings
    doc.css('div.breed-characteristics-ratings-wrapper').collect do |info|
      outer_hash = {}
      key = info.css('div.parent-characteristic').text.strip
      value = info.css('div.parent-characteristic').css('div.star').attribute('class').value[-1].to_i
      outer_hash[key] = value
      hash[:characteristics] << outer_hash
    end
    hash
  end

  def self.create_breed(url)
    info = self.breed_info(url)
    Breed.create_by_hash(info)
  end
```

I already have all the info needed, and I had created the breeds Instances, a total of 336 breeds, I didn't know there was that much. Do you?

Now is the time of display all that info. But wait a minute 336 breeds is a long list there is a better way to list that or find in that list?

Here is where I implement the Doogle(I know, not really original) search Class. Not without pain and recurring bugs everywhere, but I learned a lot challenging myself and going beyond the minimum requirements.
```
class Doogle

  attr_reader :hash, :breed

  def initialize(hash)
    @hash = hash
    puts self.doogle_logo
    input = self.ask_for_input
    self.result(input)
  end
  def search(hash)
    input = self.ask_for_input
    self.result(input)
  end

  def ask_for_input
    input = TTY::Prompt.new.ask("Type one or more characters of the desire breed.") do |q|
          q.validate /[a-zA-Z]/
        end
    input
  end

  def result(input)
    result = self.hash.select{|breed| breed.match(/^#{input}/i)}
    if !result.empty?
      breed_name, url = self.to_one_result(result)
      self.right_breed(breed_name, url)
    else
      self.not_found(input)
    end
  end

  def right_breed(breed, url)
    sure = TTY::Prompt.new.yes?("Can you confirm '#{breed}' is your desire breed?")
    if sure
      breed_object =Scraper.create_breed(url)
      @breed = breed_object
    else
      self.search(self.hash)
    end
  end

  def to_one_result(hash)
    if hash.size > 1
      hash = TTY::Prompt.new.enum_select("Select a breed", hash, per_page: 10)
      breed_name = self.hash.key(hash)
      url = self.hash[breed_name]
    else
      breed_name = hash.keys[0]
      url = hash.values[0]
    end
    return breed_name, url
  end

  def not_found(input)
    again =TTY::Prompt.new.yes?("Sorry but '#{input}'' does not match any know breed name. do you want to try again?")
    if again
      self.search(self.hash)
    end
  end

  def doogle_logo
    p = Pastel.new
    <<-logo
                        Welcome                          
                          to                             
    .#{p.blue'%%%%%'}....#{p.red('%%%%')}....#{p.yellow('%%%%')}....#{p.blue('%%%%')}...#{p.green('%%')}......#{p.red('%%%%%%')}.
    .#{p.blue('%%')}..#{p.blue('%%')}..#{p.red('%%')}..#{p.red('%%')}..#{p.yellow('%%')}..#{p.yellow('%%')}..#{p.blue('%%')}......#{p.green('%%')}......#{p.red('%%')}..... 
    .#{p.blue('%%')}..#{p.blue('%%')}..#{p.red('%%')}..#{p.red('%%')}..#{p.yellow('%%')}..#{p.yellow('%%')}..#{p.blue('%%')}.#{p.blue('%%%')}..#{p.green('%%')}......#{p.red('%%%%')}... 
    .#{p.blue('%%')}..#{p.blue('%%')}..#{p.red('%%')}..#{p.red('%%')}..#{p.yellow('%%')}..#{p.yellow('%%')}..#{p.blue('%%')}..#{p.blue('%%')}..#{p.green('%%')}......#{p.red('%%')}..... 
    .#{p.blue('%%%%%')}....#{p.red('%%%%')}....#{p.yellow('%%%%')}....#{p.blue('%%%%')}...#{p.green('%%%%%%')}..#{p.red('%%%%%%')}. 
    #{p.cyan'................................................'} 
    
    logo
  end

end
```

and that's it, wrapping everything in a CLI class, fighting again with more recurring bugs and unexpectedly returns and Nil class, and my project is ready to go.

- [Full code](https://github.com/reddevilcero/Best_Friend)
- [Video Demo](https://www.youtube.com/watch?v=f7hxouDxu2M)

If I have learned something during this project, it is to encapsulate things well and not create more instances than necessary, which leads to errors and unexpected results. I have enjoyed creating it, and I hope you do it using it or watching the video demonstration.

Happy coding
