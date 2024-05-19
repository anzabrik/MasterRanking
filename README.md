# MasterRanking: Django App that Calculates Top Books and Authors in a Topic Based on Lists Provided by User

[Video Demo](https://youtu.be/4m2JK5gQPnw).

## Built With

Django, JavaScript, Python, SQLite, BootStrap.

## Summary

MasterRanking is a web-based app that aims to help people determine most influential books and authors in a certain field of knowledge based on book lists provided by the user.

The person creates a topic and adds several book lists of their choice assigning each list a credibility score. Now, the person can view the so-called MasterRanking, where books from all the lists are ordered by rating. The rating is calculated based on several factors, including the number of occurrences of this book in the lists, the credibility of the list, the number of books in the list, the place the book has in the list.

The app does NOT impose choices: it's up to the person how they name the topic, what lists they include and what credibility score they give to each list. The person can delete/edit items, and the ranking will be recalculated.

## Models

**Master** represents the whole topic (MasterRanking), in which lists will be later added. This is the first item user creates, in a typical flow.

**List** represents a single book list, one of the many that will be added to the topic. Has ManyToMany relationship with Master (we might want to include one list into many MasterRankings).

**Author, Book**. Book has ManyToMany relationship with Author.

**Book\_In\_List**, **Author\_In\_List** connects the book or the author with a certain list. Shows the rating of this book/author in the list, the place (if it's a ranked list), and optional fields for notes. One Book can have only one Book\_In\_List in this List. One author can also have only one Author\_In\_List for this list, even if this author actually has multiple books in the list. The author\_in\_list.rating adds up ratings of all books this author has in this list.

**Book-In-Master, Author\_In\_Master** are the same as 'Book/Author\_In\_List', but for the topic. Shows the book's/author's rating and place in the topic. The book's/author's rating in a topic is the sum of their rating in all the lists in this topic.  

## Calculating The Ratings

1. As soon as the person has finished adding books to the first list, we calculate the rating of **each book in this list** via the function 'set\_bil\_rating' using one of two schemes:

  - unranked list: book\_in\_list.rating = list.credibility * 10
  - ranked list. We first count the overall list rating: list.credibility \* list.book\_in\_list\_set.count() \* 10. Then, we calculate so-called 'atom\_count' for the list (the sum of all places in the list) and for the book (depending on which place the book is on). Now, each book's rating depends on the how many 'atoms' this book has and what the rating of each atom is. Eventually, book\_in\_list.rating = book\_in\_list.atom\_count \* atom\_rating

2. The rating of each **author\_in\_list** for the new list is calculated in 'set\_ail\_rating'. We just add up the ratings of all books that this author has in this list.
3. Now, we calculate/recalculate the rating for **books and authors in the whole topic** via 'reset\_bim\_aim\_rating' by simply adding up the ratings from every list, the old ones and the new.

\*\*\*at places, we multiply/divide some of the figures by 10\*\*11 in an attempt to escape the floating point problems and make the ratings accurate.

## Using The App

Let's take an imaginary user, Mary. In search of some tips to improve her productivity, she decides to check out what useful info she can find in books on this topic. Search in 'books' in Amazon.com alone yields 50.000 results, and like almost any search result and any ranking, it is certainly biased and aimed at promoting some books over others irrespective of their real value for Mary.

To somehow counteract this, Mary uses our app to average several book lists. Let's see what she does in the app, after the registration.

1. The index page for new users displays a mini-form, where Mary types in just the name of the topic (MasterRanking). Later, when Mary adds more topics, the index page will be 'masters.html' displaying all her MasterRankings and lists.
2. Having submitted a single topic name, Mary is redirected to 'add\_list.html', where she fills in:

  - name of the list
  - topic to which the list refers (she will be able to include the list into several topics at once later, when she adds more topics)
  - list credibility. Mary gives the maximum, 5 points, to the list she got from her childhood friend, whom she also considers a productive person. A list from an obscure blog gets credibility '1'. When Mary is undecided, the credibility is automatically set to '3'.
  - number of books: determines how many empty forms for books will be displayed on the following page (automatically set to 10)
  - whether that's a ranked list
  - two optional fields: notes on the list and a link to its source

3. 'add\_books.html': Mary fills in several book forms.

  - the 'Place' column is filled in automatically in ranked lists.
  - the 'Authors' column is optional, and for every book, Mary only fills in the authors once – when she uses the same book in another list, the information will be included in author ranking automatically.
  - in the optional field 'Info', Mary might add some quotes from the book list's author, like summary or opinion – those will be displayed in the final ranking.

4. The final MasterRanking is calculated and displayed even if there's only one list (to compensate Mary for the effort), but becomes useful when more lists are added.
5. When Mary makes changes in the lists, books, or authors that could affect the rating, the figures are immediately recalculated.

## Uniqueness Constraints

Duplicate names are not allowed for topics or lists owned by the same user: the half-filled form is returned with an error message.

Book titles, however, are treated differently: when Mary adds a second, third etc. book with the same name, the book gets an index in its title ('SuperBook (1)', 'SuperBook (2)', etc.) That's because some books actually have the same name, but different authors, and they shouldn't merge. Yet, if Mary created a book with the same name by mistake, she'll see the index and will be able to delete the item.

Places in a ranked list are not unique: there can be two and more books of equal importance, and Mary will just give them the same place while adding books, and these books will have the same rating within the list.

Other users can, of course, use the same titles as Mary, and this will not interfere with her rankings. Her books, topics, lists are only hers.