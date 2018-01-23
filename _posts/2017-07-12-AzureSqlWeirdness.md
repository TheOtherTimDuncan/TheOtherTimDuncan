---
layout: post
title: "Azure Sql Weirdness"
date: 2017-07-12
disqus_identifier: 7
---
tl;dr: [Azure Sql](https://azure.microsoft.com/en-us/services/sql-database/) is a great service but can get weird. See the related [MSDN forum post](https://social.msdn.microsoft.com/Forums/en-US/dd0e54c4-df3d-4e27-884f-f306df95adc0/query-never-finishes-when-executing-alter-database-sql?forum=ssdsgetstarted).

I primarily develop [.NET](https://www.microsoft.com/net) apps that use [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-2016) as the backend. I eventually adopted the practice of executing the SQL below as part of every deployment.

```
ALTER DATABASE CURRENT SET COMPATIBILITY_LEVEL=130, ANSI_NULLS ON, ANSI_PADDING ON, ANSI_WARNINGS ON, ARITHABORT ON, CONCAT_NULL_YIELDS_NULL ON
```

About a year ago, we did our first database in [Azure Sql](https://azure.microsoft.com/en-us/services/sql-database/). I continued my practice of executing the SQL above with no problems and continued to have no problems on the additional databases we added. 

For those who might not be aware, when you create a database in Azure SQL, you have to specify the server you want to host the database. This is more of a virtual server than an actual server, and there are other nuances to the concept, but every database we created ended up on the same server. We've also been using service level S0 Standard for all our databases so far.

The other aspect to creating the server in Azure is you have to specify the master admin credentials. This is essentially the ```sa``` user for this server. It also cannot be changed after the server is created.

Around April this year I had the need to create another database. This was meant to be used by a production environment, and I decided it was past time to stop putting all our databases on the same server, especially for a production database. Since this was done before the website was ready, I just created an empty database without going through the deployment process. One thing I did that I was unsure about was use the same sa credentials from the previous server. For security reasons alone I wouldn't recommend this, but since we don't have a good way to share credentials at my shop yet - and yes, laziness as well - I reused those credentials.

It wasn't until that first real deployment that I ran into trouble. The app I was deploying used Entity Framework same as the others I have targeting Azure, and I used my [WinForms app](https://github.com/TheOtherTimDuncan/EntityFramework.DatabaseMigrator) to manage migrations and execute any SQL - like the SQL above - same as the others. Except this time the migrations were failing due to a timeout error. And then Sql Server Management Studio would slow down to an extreme crawl as well. And not just trying to execute a query. Navigating the UI was excruciatingly slow.

{% include figure.html caption="That was weird." url="https://media.giphy.com/media/3uyIgVxP1qAjS/giphy.gif" %}

This went on over several deployments over the next couple months. I checked everything I could think of, compared configurations, and made an even deeper dent in the edge of desk but nothing worked. The only possible reason I could think of was reusing those credentials. It didn't make sense, but I at least had to rule it out. But since you can't change the credentials, that meant deleting that server - and database - and recreating the server.

Ugh.

At least I could copy the database someplace else then copy it back. So I got a maintenance window and did just that. Once everything was done, I tried that query again.

{% include figure.html caption="Nope." url="https://media.giphy.com/media/3uyIgVxP1qAjS/giphy.gif" %}

Now I'm desperate. I reach out via [Twitter](https://twitter.com/TheOtherTDuncan/status/883782839807901696) and get a quick response from Azure Support who then ask me to post it on an [MSDN forum](https://social.msdn.microsoft.com/Forums/en-US/dd0e54c4-df3d-4e27-884f-f306df95adc0/query-never-finishes-when-executing-alter-database-sql?forum=ssdsgetstarted).

So yes, the query did eventually finish. It looks like it took about an hour and a half though. And then Azure's record of that query disappears the following day...and all of a sudden the query starts working.

So I don't know. Maybe it was reusing the credentials. Maybe some back end glitch by the Azure infrastructure with the first server instance was fixed by re-creating the server. And maybe I had to wait several hours for the bits to finish moving around. Maybe I did something to piss off the hamsters driving the server the first time around and didn't the second time around. Maybe I should just accept it...

{% include figure.html caption="Just accept it." url="https://media.giphy.com/media/12NUbkX6p4xOO4/giphy.gif" %}

I think I'll start using this SQL against every database I create as a test. And if it gives me a problem, start over. But let's hope this was just a one-time thing.

Corrections: spelling, grammar