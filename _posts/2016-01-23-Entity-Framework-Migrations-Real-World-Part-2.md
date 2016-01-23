---
layout: post
title:  "Entity Framework Migrations In The Real World, Part 2"
date: 2016-01-23
disqus_identifier: 5
---
As I mentioned in [part 1](/archive/Entity-Framework-Migrations-Real-World/), when I first started using Entity Framework migrations, I wasn't able to find any examples of anyone using them in real world scenarios. Everything I found was usually a simplistic example just to demonstrate a concept.

When I was discussing Entity Framework migrations with a skeptical senior developer recently and mentioned the above, he asked if maybe there was a reason for that implying that the reason was because nobody actually used them. While I find it difficult to believe that nobody uses migrations, I also realized he had raised a good question.

But that raised an even more difficult question. How do I find out who uses migrations and how? StackOverflow isn't the best place for this type of question. I don't do conventions and don't have any significant connections to the developer community. I'm one of those [dark matter developers](http://www.hanselman.com/blog/DarkMatterDevelopersTheUnseen99.aspx) that only comes out of my hole for food. Or when my wife starts making some pointed comments. I finally realized there was only one place to go. A place I have said I would never go...

Twitter.

For those who don't know me, I'm from the generation before social media. If you want to date me (please don't), I remember using an Apple IIe. I'm also the typical computer geek that prefers hanging out with a computer instead of humans. I can barely handle LinkedIn. So if you had this unsettling feeling today that reality as you know it has changed and will never be the same, I'm sorry. My bad.

So enough with the rambling preamble. These are the questions that I'm looking to find answers for:

1. Who uses Entity Framework Migrations for applications that are currently in production?
2. How do you apply those migrations during a production release? Do you generate a SQL script? Or wave a magic wand?
3. If you are using Entity Framework but aren't using migrations, what do you use to source control your database? And what drove your decision to not use migrations?

I'm going to put this out on Twitter and update this post with the replies I find interesting. And of course, comments are encouraged and appreciated.