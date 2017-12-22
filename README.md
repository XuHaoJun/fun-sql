# fun-sql

A functional programming style SQL query builder.

## Why?

For fun with SQL.

## Example

```javascript
const funSqlBuild = require("fun-sql/build");

const query = funSqlBuild((funSql) => {
    const BookSet = funSql.table("books");
    const Book = BookSet.fields;

    const AuthorSet = funSql.table("authors");
    const Author = AuthorSet.fields;

    const BookAuthorSet = funSql.table("bookAuthors");
    const BookAuthor = BookAuthorSet.fields;

    const joined = BooksSet
        .intersection(BookAuthorSet, () => {
            return Book.id === BookAuthor.bookId;
        })
        .intersection(AuthorSet, () => {
            return BookAuthor.authorId === Author.id;
        });
    // const leftoined = Books.intersection(Authors, () => ...).union(Authors);
    // const rightJoined = Books.intersection(Authors, () => ...).union(Authors);
    // const fullJoined = Books.union(Authors);

    const grouped = joined.groupBy(Author.id);

    const mapped = grouped.map(() => {
        const bookId = Book.id;
        const bookTitle = Book.title;
        const authorName = Author.name;
        const totalSales = funSql.sum(Book.sales);
        const avgSales = funSql.avg(Book.sales);
        return { bookId, bookTitle, authorId, authorName, totalSales, avgSales };
    });

    const Mapped = mapped.fields;

    const ordered = mapped.orderBy(
                    { field: Mapped.totalSales, sort: "desc" },
                    { field: Book.createdAt }
                  );

    return ordered;
});

console.log(query.toString());
// SELECT Book.authorId as authorId,
//        Book.id as bookId,
//        Author.name as authorName,
//        SUM(Book.sales) as totalSales,
//        SUM(Book.sales) as avgSales,
// FROM books as Book
// INNER JOIN bookAuthors as BookAuthor ON Book.id = BookAuthor.bookId
// INNER JOIN authors as Author ON BookAuthor.authorId = Author.id
// GROUP BY Author.id
// ORDER BY totalSales


const funSqlTestExec = require("fun-sql/testExec");
const dataSets = {
    books: [
        {id: 1, title: "Structure and Interpretation of Computer Programs", sales: 5169},
        {id: 2, title: "Hacking the Xbox", sales: 574},
    ],
    bookAuthors: [
        {bookId: 1, authorId: 1},
        {bookId: 1, authorId: 2},
        {bookId: 1, authorId: 3},
        {bookId: 2, authorId: 1},
    ],
    authors: [
        {id: 1, name: "Harold Abelson"},
        {id: 2, name: "Gerald Jay Sussman"},
        {id: 3, name: "Julie Sussman"},
    ]
};

const rows = funSqlTestExec({dataSets: dataSets, query: query2});
console.log(rows);
// [
//   {
//     bookId: 1,
//     authorName: "Harold Abelson",
//     ...
//   },
//   {
//     bookId: 1,
//     authorName: "Gerald Jay Sussman",
//     ...
//   },
//   ...
// ]
```

## How it work?

```javascript
const ast = parse(String(yourBuildFunction));
const sql = ast.toSQL();
```

## Limitation

Can't work with javascript compressor like UglifyJS, but you can exclude *.funSql.js files.
