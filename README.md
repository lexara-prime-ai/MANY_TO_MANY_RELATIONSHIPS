# Entity Framework Core: Many-to-Many Relationships - In-Depth Documentation
(Not the kind you're thinking about)
## Table of Contents

1.  Introduction
2.  Setting Up Your Environment
3.  Defining Entities and DbContext
4.  Configuring Many-to-Many Relationships
5.  Performing CRUD Operations
6.  Querying Many-to-Many Relationships
7.  Conclusion

## 1. Introduction

**Entity Framework Core** (EF Core) is an **object-relational mapping** (ORM) framework that simplifies database interactions in .NET applications. **Many-to-many** relationships are common in databases where entities from two different tables have a many-to-many association. In this documentation, we'll explore how to work with many-to-many relationships in ***EF Core***.

## 2. Setting Up Your Environment

Install EF Core using the following commands :

```powershell
C:\> dotnet add package Microsoft.EntityFrameworkCore
```
```powershell
C:\> dotnet add package Microsoft.EntityFrameworkCore.SqlServer` 
```

## 3. Defining Entities and DbContext

Let's say we're building a simple blogging application with **Authors** and **Posts**. An Author can write ***multiple Posts***, and a Post can have ***multiple Authors***. Here's how we would define your entities and DbContext:

```csharp

public class Author
{
    public int AuthorId { get; set; }
    public string Name { get; set; }
    public ICollection<PostAuthor> PostAuthors { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public ICollection<PostAuthor> PostAuthors { get; set; }
}

public class PostAuthor
{
    public int PostId { get; set; }
    public Post Post { get; set; }
    public int AuthorId { get; set; }
    public Author Author { get; set; }
}

public class BlogDbContext : DbContext
{
    public DbSet<Author> Authors { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<PostAuthor>().HasKey(pa => new { pa.PostId, pa.AuthorId });
    }
}
```

## 4. Configuring Many-to-Many Relationships

In the example above, we've introduced a join entity `PostAuthor` to represent the many-to-many relationship between `Author` and `Post`. The `OnModelCreating` method configures the composite primary key for the join entity.

## 5. Performing CRUD Operations

### Adding Authors and Posts:

```csharp
using (var context = new BlogDbContext())
{
    var author1 = new Author { Name = "John Doe" };
    var author2 = new Author { Name = "Jane Smith" };
    
    var post1 = new Post { Title = "Introduction to EF Core" };
    var post2 = new Post { Title = "Advanced Querying in EF Core" };
    
    context.AddRange(author1, author2, post1, post2);
    context.SaveChanges();
    
    var postAuthor1 = new PostAuthor { Post = post1, Author = author1 };
    var postAuthor2 = new PostAuthor { Post = post1, Author = author2 };
    
    context.AddRange(postAuthor1, postAuthor2);
    context.SaveChanges();
}
```

### Updating Authors and Posts:

```csharp
`using (var context = new BlogDbContext())
{
    var post = context.Posts.Include(p => p.PostAuthors).FirstOrDefault(p => p.Title == "Introduction to EF Core");
    var author = context.Authors.FirstOrDefault(a => a.Name == "John Doe");
    
    post.PostAuthors.Add(new PostAuthor { Author = author });
    context.SaveChanges();
}
```

### Deleting Authors and Posts:

```csharp
using (var context = new BlogDbContext())
{
    var post = context.Posts.FirstOrDefault(p => p.Title == "Advanced Querying in EF Core");
    context.Posts.Remove(post);
    context.SaveChanges();
}
```

## 6. Querying Many-to-Many Relationships

### Getting Authors of a Post:

```csharp
using (var context = new BlogDbContext())
{
    var post = context.Posts.Include(p => p.PostAuthors).ThenInclude(pa => pa.Author).FirstOrDefault();
    foreach (var postAuthor in post.PostAuthors)
    {
        Console.WriteLine(postAuthor.Author.Name);
    }
}
```

### Getting Posts of an Author:

```csharp
using (var context = new BlogDbContext())
{
    var author = context.Authors.Include(a => a.PostAuthors).ThenInclude(pa => pa.Post).FirstOrDefault();
    foreach (var postAuthor in author.PostAuthors)
    {
        Console.WriteLine(postAuthor.Post.Title);
    }
}
```

## 7. Conclusion
Working with ***many-to-many*** relationships in Entity Framework Core involves defining join entities, configuring relationships, and performing CRUD operations accordingly.
