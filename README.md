# fun-sql

Another SQL query builder.

## Why?

For fun with SQL.

## Example

```javascript
const funSqlBuild = require('fun-sql/build');

const query = funSqlBuild(funSql => {
  const BookSet = funSql.table('books');
  const Book = BookSet.fields;

  const AuthorSet = funSql.table('authors');
  const Author = AuthorSet.fields;

  const BookAuthorSet = funSql.table('bookAuthors');
  const BookAuthor = BookAuthorSet.fields;

  const joined1 = BooksSet.intersection(BookAuthorSet, () => {
    return Book.id === BookAuthor.bookId;
  }).intersection(AuthorSet, () => {
    return BookAuthor.authorId === Author.id;
  });
  const AnotherSet = () => {
    return BookSet.filter(() => {
      return Book.authorId !== null;
    }).map(() => {
      const { title } = Book;
      return { bookTitle: title };
    });
  }();
  const joined = joined1.leftJoin(AnotherSet);

  const filted = joined.filter(() => {
    return Book.sales > 100 && funSql.len(Book.title) > 7;
  });

  const grouped = filted.groupBy(Author.id);

  const mapped = grouped.map(() => {
    const bookId = Book.id;
    const bookTitle = Book.title;
    const authorName = Author.name;
    const totalSales = funSql.sum(Book.sales);
    let range;
    if (totalSales === 0) {
      range = 'none';
    } else if (totalSales > 1000) {
      range = 'good';
    } else {
      range = 'normal';
    }
    const avgSales = funSql.avg(Book.sales);
    return {
      bookId,
      bookTitle,
      authorId,
      authorName,
      totalSales,
      avgSales,
      range
    };
  });

  const Mapped = mapped.fields;

  const ordered = mapped.orderBy(
    { field: Mapped.totalSales, sort: 'desc' },
    { field: funSql.len(Mapped.authorName) }
  );

  return ordered;
});

console.log(query.toString());
// SELECT Book.id as bookId,
//        Book.title as bookTitle,
//        Book.authorId as authorId,
//        Author.name as authorName,
//        SUM(Book.sales) as totalSales,
//        AVG(Book.sales) as avgSales,
//        CASE WHEN totalSales = 0 THEN 'none'
//             WHEN totalSales > 1000 THEN 'good'
//             ELSE 'normal' AS range
// FROM books as Book
// INNER JOIN bookAuthors as BookAuthor ON Book.id = BookAuthor.bookId
// INNER JOIN authors as Author ON BookAuthor.authorId = Author.id
// LEFT JOIN
// (
//   SELECT bookTitle as Book.title
//   FROM books as Book
//   WHERE Book.authorId IS NOT NULL
// ) AS AnotherSet
// WHERE Book.sales > 100 and LEN(Book.title) > 7
// GROUP BY Author.id
// ORDER BY totalSales DESC, LEN(authorName)

const funSqlSimulate = require('fun-sql/simulate');
const dataSets = {
  books: [
    {
      id: 1,
      title: 'Structure and Interpretation of Computer Programs',
      sales: 5169
    },
    { id: 2, title: 'Hacking the Xbox', sales: 574 }
  ],
  bookAuthors: [
    { bookId: 1, authorId: 1 },
    { bookId: 1, authorId: 2 },
    { bookId: 1, authorId: 3 },
    { bookId: 2, authorId: 1 }
  ],
  authors: [
    { id: 1, name: 'Harold Abelson' },
    { id: 2, name: 'Gerald Jay Sussman' },
    { id: 3, name: 'Julie Sussman' }
  ]
};

const rows = funSqlSimulate.exec({ dataSets: dataSets, query: query2 });
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

## etc.
### from
``` js
funSql.table("table1", "table2");
```

### where
``` js
Book.filter(() => {
  // ...
});
```

### select
``` js
Book.map(() => {
  // ...
});
```
### switch
``` js
let range;
if (totalSales === 0) {
  range = 'none';
} else if (totalSales > 1000) {
  range = 'good';
} else {
  range = 'normal';
}
```


## How it work?
### to SQL
```javascript
const ast = parse(String(yourBuildFunction));
const sql = ast.toSQL();
```
### Simulate
Iter callback n times and change code from `Book.id === BookAuthor.bookId` to `Book.next().id === BookAuthor.next().bookId`.

## Limitation

Can't work with javascript compressor like UglifyJS, but you can exclude `*.funSql.js files`.

## Thinking SQL
Why everything is flatten?
It make more low level manipulation compare C language Pointer,
normalization make more table and you must store redundant KEYs and link theme loop again and again....


Why use Declarative language?
Imperative language with good static analysis is not enough?
