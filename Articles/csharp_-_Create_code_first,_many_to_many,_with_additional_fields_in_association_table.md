csharp_-_Create_code_first,_many_to_many,_with_additional_fields_in_association_table

[Source](http://stackoverflow.com/questions/7050404/create-code-first-many-to-many-with-additional-fields-in-association-table)

# c# - Create code first, many to many, with additional fields in association table

It's not possible to create a many-to-many relationship with a customized join table. In a many-to-many relationship EF manages the join table internally and hidden. It's a table without an Entity class in your model. To work with such a join table with additional properties you will have to create actually two one-to-many relationships. It could look like this:


```
public class Member
{
    public int MemberID { get; set; }

    public string FirstName { get; set; }
    public string LastName { get; set; }

    public virtual ICollection<MemberComment> MemberComments { get; set; }
}

public class Comment
{
    public int CommentID { get; set; }
    public string Message { get; set; }

    public virtual ICollection<MemberComment> MemberComments { get; set; }
}

public class MemberComment
{
    [Key, Column(Order = 0)]
    public int MemberID { get; set; }
    [Key, Column(Order = 1)]
    public int CommentID { get; set; }

    public virtual Member Member { get; set; }
    public virtual Comment Comment { get; set; }

    public int Something { get; set; }
    public string SomethingElse { get; set; }
}
```
If you now want to find all comments of members with `LastName` = "Smith" for example you can write a query like this:


```
var commentsOfMembers = context.Members
    .Where(m => m.LastName == "Smith")
    .SelectMany(m => m.MemberComments.Select(mc => mc.Comment))
    .ToList();
```
...or...


```
var commentsOfMembers = context.MemberComments
    .Where(mc => mc.Member.LastName == "Smith")
    .Select(mc => mc.Comment)
    .ToList();
```
Or to create a list of members with name "Smith" (we assume there is more than one) along with their comments you can use a projection:


```
var membersWithComments = context.Members
    .Where(m => m.LastName == "Smith")
    .Select(m => new
    {
        Member = m,
        Comments = m.MemberComments.Select(mc => mc.Comment)
    })
    .ToList();
```
If you want to find all comments of a member with `MemberId` = 1:


```
var commentsOfMember = context.MemberComments
    .Where(mc => mc.MemberId == 1)
    .Select(mc => mc.Comment)
    .ToList();
```
Now you can also filter by the properties in your join table (which would not be possible in a many-to-many relationship), for example: Filter all comments of member 1 which have a 99 in property `Something` :


```
var filteredCommentsOfMember = context.MemberComments
    .Where(mc => mc.MemberId == 1 && mc.Something == 99)
    .Select(mc => mc.Comment)
    .ToList();
```
Because of lazy loading things might become easier. If you have a loaded `Member` you should be able to get the comments without an explicite query:


```
var commentsOfMember = member.MemberComments.Select(mc => mc.Comment);
```
I guess that lazy loading will fetch the comments automatically behind the scenes.

**Edit**

Just for fun a few examples more how to add entities and relationships and how to delete them in this model:

1) Create one member and two comments of this member:


```
var member1 = new Member { FirstName = "Pete" };
var comment1 = new Comment { Message = "Good morning!" };
var comment2 = new Comment { Message = "Good evening!" };
var memberComment1 = new MemberComment { Member = member1, Comment = comment1,
                                         Something = 101 };
var memberComment2 = new MemberComment { Member = member1, Comment = comment2,
                                         Something = 102 };

context.MemberComments.Add(memberComment1); // will also add member1 and comment1
context.MemberComments.Add(memberComment2); // will also add comment2

context.SaveChanges();
```
2) Add a third comment of member1:


```
var member1 = context.Members.Where(m => m.FirstName == "Pete")
    .SingleOrDefault();
if (member1 != null)
{
    var comment3 = new Comment { Message = "Good night!" };
    var memberComment3 = new MemberComment { Member = member1,
                                             Comment = comment3,
                                             Something = 103 };

    context.MemberComments.Add(memberComment3); // will also add comment3
    context.SaveChanges();
}
```
3) Create new member and relate it to the existing comment2:


```
var comment2 = context.Comments.Where(c => c.Message == "Good evening!")
    .SingleOrDefault();
if (comment2 != null)
{
    var member2 = new Member { FirstName = "Paul" };
    var memberComment4 = new MemberComment { Member = member2,
                                             Comment = comment2,
                                             Something = 201 };

    context.MemberComments.Add(memberComment4);
    context.SaveChanges();
}
```
4) Create relationship between existing member2 and comment3:


```
var member2 = context.Members.Where(m => m.FirstName == "Paul")
    .SingleOrDefault();
var comment3 = context.Comments.Where(c => c.Message == "Good night!")
    .SingleOrDefault();
if (member2 != null && comment3 != null)
{
    var memberComment5 = new MemberComment { Member = member2,
                                             Comment = comment3,
                                             Something = 202 };

    context.MemberComments.Add(memberComment5);
    context.SaveChanges();
}
```
5) Delete this relationship again:


```
var memberComment5 = context.MemberComments
    .Where(mc => mc.Member.FirstName == "Paul"
        && mc.Comment.Message == "Good night!")
    .SingleOrDefault();
if (memberComment5 != null)
{
    context.MemberComments.Remove(memberComment5);
    context.SaveChanges();
}
```
6) Delete member1 and all its relationsships to the comments:


```
var member1 = context.Members.Where(m => m.FirstName == "Pete")
    .SingleOrDefault();
if (member1 != null)
{
    context.Members.Remove(member1);
    context.SaveChanges();
}
```
This deletes the relationships in `MemberComments` too because the one-to-many relationships between `Member` and `MemberComments` and between `Comment` and `MemberComments` are setup with cascading delete by convention. And this is the case because `MemberId` and `CommentId` in `MemberComment` are detected as foreign key properties for the `Member` and `Comment` navigation properties and since the FK properties are of type non-nullable `int` the relationship is required which finally causes the cascading-delete-setup. Makes sense in this model, I think.

  
  
  
  
