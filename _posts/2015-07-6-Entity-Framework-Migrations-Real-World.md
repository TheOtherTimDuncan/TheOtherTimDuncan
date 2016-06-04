---
layout: post
title: "Entity Framework Migrations In The Real World"
date: 2015-07-26
disqus_identifier: 2
---
tl;dr: The nuget package can be found [here](http://www.nuget.org/packages/EntityFramework.DatabaseMigrator/). Source code with a readme can be found [here](https://github.com/TheOtherTimDuncan/EntityFramework.DatabaseMigrator).

My first introduction to [Entity Framework](http://www.asp.net/entity-framework) was in 2012 with version 4. I hated it. It didn't help that I was relatively new to C#, and that the codebase I was working in wasn't the greatest, but dealing with that XML-based EDMX model was a nightmare.

Updating the database model was always a pain. I tried a couple times to update the model by manually editing the XML, but there was always something I missed that would cause it to fail. What made things even more complicated was the entire database schema was not in the XML model yet. That meant I had to let the model update itself from the schema in the database, diff the changes and then take out what wasn't supposed to be in the model yet.

Add that fun to the joy of ProviderManifestToken. We ran into a bug once where the Entity Framework model was trying to make the DateTime properties a datetime2 in the database. Since the majority of our customers were still on SQL Server 2005, that didn't work so well. We eventually figured out that we had to change the ProviderManifestToken to 2005, but since there was nothing in the UI to set that value, it had to be done manually. And always remembered to be done again every time the model was updated since Entity Framework would always flip it back to 2008.

That experience led me to write my own ORM. When I started on that, I was aware of the general consensus that trying to design your own ORM was an act of hubris. Since the codebase was still in .NET 3.5, and the budget and permission for 3rd party alternatives was pretty much none, I decided to do it anyway. I learned a lot along the way and was pretty happy with the end result. The one feature I never tried to add and that I started to want was support for LINQ. My ORM used expressions heavily to allow building queries in a strongly-typed fashion, but going the next step and adding LINQ support wasn't worth the effort.  

Especially since by this time, Entity Framework 6.0 had been released. I had read enough about it to think it had potential, and the Code First approach looked like exactly what I was looking for. In late 2014, I finally had a chance to start using it in a project and started migrating that codebase from my ORM to Entity Framework. There were some pain points, and some dents in my forehead, but in the end, I was sold. Entity Framework 6.0 was definitely the way to go.
 
One of my favorite features in Entity Framework was [migrations](https://msdn.microsoft.com/en-us/data/jj591621.aspx). I had started using database projects in Visual Studio in 2012 and loved them, but I liked the idea of deriving the schema from the data model and not having to maintain and keep in sync both the database schema and the data model. Late in 2013, I had also learned one of the weaknesses of database projects - they require the ability to remotely access the database server. And for the record, I'm not installing Visual Studio on a production server. In 2013, one of the projects I was using database projects with moved to a new hosting provider, and I no longer had VPN access to the staging or production servers. That meant having to download a backup of the database any time I wanted to compare the database project to production. Rather annoying.
 
But the more I read about migrations, the more uncertain I was. Every demo or example I read used the Powershell console in Visual Studio to execute or rollback migrations. I couldn't find any good examples of how to handle migrations when deploying to production. You can find a couple answers on Stack Overflow like this [one](http://stackoverflow.com/questions/26384290/entity-framework-migrations-i-dont-have-access-to-production-sql-server), but that kind of approach didn't really suit me. Too manual and too easy to make mistakes. And turning on automatic migrations was a definite not-gonna-happen. There was no way I was going to allow a migration to be executed without an easy way to rollback or an easy way to review the generated SQL.
 
Eventually, I found [this post](http://romiller.com/2012/02/09/running-scripting-migrations-from-code/) from Rowan Miller, and [this post](http://whiteknight.github.io/2013/01/26/efcodeonlymigrations.html) from Andrew Whitworth. Those two posts gave me some ideas on how to deal with production deployments and inspired me to write a WinForms utility that I could copy to the production server and use from there.
 
Which finally allows me to get to the point of all this. After I first created that utility, I used it in another project by copying the code. When I had reason to use it in a third project, I decided it was time to refactor it to make it easier to reuse. And that also gave me the opportunity to wrap it up in a [nuget package](http://www.nuget.org/packages/EntityFramework.DatabaseMigrator/) and share my [genius](https://github.com/TheOtherTimDuncan/EntityFramework.DatabaseMigrator) with everyone.
 
 You may declare your eternal gratitude in the comments below.
 
I assume some instructions would be appreciated as well. I will assume you already have Entity Framework working in your project and already have migrations enabled. If you need help getting started with migrations, please start with [Microsoft's tutorial](https://msdn.microsoft.com/en-us/data/jj591621.aspx), or ask your question in the comments. An [example project](https://github.com/TheOtherTimDuncan/EntityFramework.DatabaseMigrator/tree/master/EntityFramework.DatabaseMigrator.Example) is also available.
 
1. Add a new WinForms project to your solution. For the sake of these instructions, I will name it DatabaseMigrator.
{% include figure.html caption="Add Windows Forms project" url="/images/posts/Entity-Framework-Migrations-Real-World/Add Project.png" %}
2. Delete the form automatically added to the new project.
3. Install the EntityFramework.DabaseMigratorPackage

```powershell
PM> Install-Package EntityFramework.DatabaseMigrator
```

4. Open `program.cs` in the root of your project and change Form1 to EntityFramework.DatabaseMigrator.DatabaseMigrator

```csharp
static class Program
{
    /// <summary>
    /// The main entry point for the application.
    /// </summary>
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new EntityFramework.DatabaseMigrator.DatabaseMigrator());
    }
}
```

5. Change your migration configuration to inherit from [BaseMigrationConfiguration](https://github.com/TheOtherTimDuncan/EntityFramework.DatabaseMigrator/blob/master/EntityFramework.DatabaseMigrator/Migrations/BaseMigrationConfiguration.cs) or add and implement the [IMigrationConfiguration](https://github.com/TheOtherTimDuncan/EntityFramework.DatabaseMigrator/blob/master/EntityFramework.DatabaseMigrator/Migrations/IMigrationConfiguration.cs) interface.

```csharp
internal sealed class Configuration : BaseMigrationConfiguration<EntityFramework.DatabaseMigrator.Example.Data.BlogContext>
{
    public Configuration()
    {
        AutomaticMigrationsEnabled = false;
        ContextKey = "EntityFramework.DatabaseMigrator.Example.Data.BlogContext";
    }
}
```

6. Start 'er up!
{% include figure.html caption="Database Migrator" url="/images/posts/Entity-Framework-Migrations-Real-World/Database Migrator.png" %}

The UI should be self-explanatory. If you would rather roll your own UI, all you need to do is inherit your form from [BaseDatabaseMigrator](https://github.com/TheOtherTimDuncan/EntityFramework.DatabaseMigrator/blob/master/EntityFramework.DatabaseMigrator/BaseDatabaseMigrator.cs). Use the [Database Migrator](https://github.com/TheOtherTimDuncan/EntityFramework.DatabaseMigrator/blob/master/EntityFramework.DatabaseMigrator/DatabaseMigrator.cs) form as an example of how to wire up the base class to your form.

As a bonus, pay attention to the SQL generated by a pending migration and notice something - it includes the SQL to insert the row into the __MigrationHistory table. Why does that matter? 

I've seen a few questions about how to get the first migration into the database so future migrations can be executed. The most common answer is to create an empty migration just for the purpose of getting that base model into the __MigrationHistory table. The problem with that is it means you can't ever have a real Create migration. Something else I've realized as I've worked with migrations is you'll have this problem multiple times because you will eventually want to delete all your migrations and start fresh with a new initial Create.

So what do I do? After I create - or re-create - my initial Create migration, I view the SQL, copy the query to insert the row into __MigrationHistory into SQL Server Management Studio, then execute the query from there.

<!-- Thanks to http://ateamresource.com/gallery.php?action=view_image&id=107 -->
{% include figure.html caption="I love it when a plan comes together." url="/images/posts/Entity-Framework-Migrations-Real-World/Hannibal Smith.jpg" %}
