# Learning GraphQL from NetNinja
In anticipation of my upcoming internship with [Axio](https://axio.com/careers/?gh_jid=4020412002) in Spring 2020, I've started learning GraphQL to minimize the ramp-up time I'll need when I start working there.

## Why do you want to work at Axio?
I had a brief meeting with their [VP of Technology](https://axio.com/leadership/), got to meet the team and see what their work environment was like, and would really love the opportunity to start my career there as a junior developer. They've hired devs from bootcamps in the past and the role was recommended to me by a former intern.

# GraphQL API
In this project, you can use GraphQL mutations to add authors and their books to a MongoDB database with two collections: authors and books. Through Graphiql, you can use two mutations to add new documents for a book or an author. 

All fields needed to add new authors and books are non-null types, so you cannot add new documents to the database if something is missing from your mutation.

## Types

There are two types (and corresponding MongoDB models): BookType and AuthorType.

```
const BookType = new GraphQLObjectType({
  name: 'Book',
  fields: () => ({
    id: { type: GraphQLID },
    name: { type: GraphQLString },
    genre: { type: GraphQLString },
    author: {
      type: AuthorType,
      resolve(parent) {
        return Author.findById(parent.authorID); 
      }
    }
  })
});

const AuthorType = new GraphQLObjectType({
  name: 'Author',
  fields: () => ({
    id: { type: GraphQLID },
    name: { type: GraphQLString },
    age: { type: GraphQLInt },
    books: {
      type: new GraphQLList(BookType),
      resolve(parent) {
        return Book.find({ authorID: parent.id });
      }
    }
  })
});


```

### Root Query
The root query allows you to find a specific book or author, or return a list of all books or all authors. Specific properties can be requested at will, which is what makes GraphQL so great!

```
const RootQuery = new GraphQLObjectType({
  name: 'RootQueryType',
  fields: {
    book: {
      type: BookType,
      args: { id: { type: GraphQLID } },
      resolve(parent, args) {
        return Book.findById(args.id);
      }
    },
    author: {
      type: AuthorType,
      args: { id: { type: GraphQLID } },
      resolve(parent, args) {
        // return _.find(authors, { id: args.id });
        return Author.findById(args.id);
      }
    },
    books: {
      type: new GraphQLList(BookType),
      resolve() {
        // return books;
        return Book.find({});
      }
    },
    authors: {
      type: new GraphQLList(AuthorType),
      resolve() {
        // return authors;
        return Author.find({});
      }
    }
  }
});
```

#### Root Query Example
Return all books and for each book, show the name and age of the author.

```
{
  books {
    name
    genre
    author {
      name
      age
    }
  }
}
```

## Mutations

### Adding an author
An entry for a new author takes two fields: name (string) and age (number).

```
mutation {
  addAuthor(name: "China Miéville" age: 47) {
    age
    name
  }
}
```

This will create a new author with a name and age (and a unique ID) and then return the name and age of the author you just created. An ID will be created and assigned to the document as well. **Don't forget: in GraphiQL, you must use double quotes for strings.**

Before creating a book, you must grab the authorID from that document in your database. 

### Adding a book
Adding a new document for a book requires a name (string), genre (string), and the appropriate authorID (string). All fields are non-null

```
mutation {
  addBook(name: "Embassytown", genre: "Science Fiction", authorID: "5dec37268919bd29d08bec39") {
    name
    genre
  }
}
```

This will add a new book, then return the name and genre.
