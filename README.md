# Project 2

### 1. The name and UNI of both teammates.

Ruoyang Liu: rl3323

Kyurang Kang: kk3583

  

### 2. The name of the PostgreSQL account where your database is on our server.

kk3583

  

### 3. New items added:

#### a. text attribute

We add a text attribute `storyline` in the table `Contents.` This attribute holds a long textual description of the storyline of each movie or show. We want to add this attribute because in addition to the genres of a movie or show, we may want to find celebrities who have acted in a movie with or as some specific elements or roles, for example with a dragon, or as a family member.

  

Here is an example scenario and query that involve this new attribute:

For example, if we want to find all celebrities who have acted in some contents involving dealing with family members, we can use the following query to do a full-text search in the ‘storyline’ attribute.

  
```
SELECT C.cid, C.name

FROM Contents Con, Appeared_In_Movies A, Celebrities C

WHERE Con.conid = A.conid AND A.cid = C.cid AND to_tsvector(Con.storyline) @@ to_tsquery('family | parent | parents | father | mother' )

UNION

SELECT C.cid, C.name

FROM Contents Con, Appeared_In_Shows A, Celebrities C

WHERE Con.conid = A.conid AND A.cid = C.cid AND to_tsvector(Con.storyline) @@ to_tsquery('family | parent | parents | father | mother' );
```
  

This will give us the output as below, which are the celebrities id and their name who satisfies our condition.

  
```
cid | name

-----+------------------------

25 | Matt Smith

63 | Mandy Moore

34 | Kikunosuke Toya

42 | Jeremy Allen White

62 | Ellar Coltrane

24 | Rhys Ifans

7 | Michelle Yeoh

40 | Saoirse-Monica Jackson

35 | Tomori Kusunoki

41 | Louisa Harland

21 | Jurnee Smollett

50 | Shawn Mendes

9 | Javier Bardem

10 | Scoot McNairy

64 | Amanda Seyfried

8 | Stephanie Hsu

20 | Allison Janney

61 | Ethan Hawke

43 | Ebon Moss-Bachrach

(19 rows)
```
  
  

#### b. Array attribute

We add an array attribute `first_4_week_box_office_domestic` in the table `Movies.` This attribute holds an array of 4 real numbers which shows the first 4 week of the movie's domestic box office upon its release. We want to add this attribute because in addition to the total worldwide box office of a movie, we are also interested in seeing if the movie got more and more popular after it released or it faded quickly in the movie market. This characteristic can help us find staffs that have produced high quality movies that stays popular over the first month of it release.

  

Here is an example scenario and query that involve this new attribute:

For example, if we want to find all directors who have directed a movie that stayed popular during the first month of its release. Since some movies are released at the end of a week, we used the difference between the 4th week's domestic box office and the maximum of the first and the second week's domestic box office to get the relative change. Here, we want the drop to be less than 50%p. The query is as follow:

  
```
SELECT S.sid, S.name

FROM Movies M, Directors D, Directed Dir, Staffs S, (SELECT conid, (SELECT MAX(x) FROM unnest(first_4_week_box_office_domestic[1:2]) x) AS max_1or2 FROM Movies) AS Max_box

WHERE D.sid = Dir.sid AND M.conid = Dir.conid AND S.sid = D.sid AND Max_box.conid = M.conid

AND ( Max_box.max_1or2 - M.first_4_week_box_office_domestic[1] ) / Max_box.max_1or2 > -0.5;
```
  

We get a list of output directors with their staff id.

  
```
sid | name

-----+--------------------

14 | David Leitch

21 | Jaume Collet-Serra

40 | Taika Waititi

5 | Baz Luhrmann

15 | David O. Russell

12 | David Gordon Green

10 | Dan Kwan

11 | Daniel Scheinert

15 | David O. Russell

47 | Adam McKay

49 | Martin Scorsese

51 | Richard Linklater

52 | Adam Shankman

54 | Nathan Greno

56 | Phyllida Lloyd

58 | Tom Hooper

(16 rows)
```
  

#### c. Trigger

Finally, we added a trigger to check the attributes of contents data when user insert new rows to `Contents` table or update attributes to the existing rows. Since there are many constraints on the attribute of contents, we want to check them all using the trigger before inserting them into our table. Bad rows including having a `Null` value for `title`, having a `release_date` later than today's date, having a `motion picture rate (mpr)` other than the predefined ones, having an `imdb_rating` out of range 0 to 10, and having a negative number for `num_of_reviews`. When inserting a new row or update an attribute of existing row in the table `Contents` that has any one of the above bad conditions will trigger our trigger and the trigger will reject the insertion or update. Otherwise, the insertion and update will be successful.

  

Here are some example queries that our trigger will reject the action.

  

a `NULL` title:

`INSERT INTO Contents VALUES (100,NULL,'2000-2-2','R',5,100);`

**![](https://lh3.googleusercontent.com/Kri-WR2R2fzPaUEZkRjtnXPo3yt8znkDX4ZRsr9NkB2UcgsCaAn5nInGLpArV0x5FJL_XYSLjA_hv5HJb4_eM9DjFYi2uqU9zB-RGsTKiCh7YnyfH3lhpDxOUEa2k6dakC1Ah7hwG82L8hweGaZhuDdXTQkcgxyyVxU_0hrG5jS7MPtvsFA3y-bGAXNOXw)**

an invalid `release_date`:
`UPDATE Contents SET release_date = '2023-01-01' where conid = 1;`
**![](https://lh4.googleusercontent.com/mi9cW27ywI1MdxzqCdVzoAtyMq-ArZXZ9u18lrGaeQRLaWUs2geIA3kMoKzbLkb9wMpqcOOkTHwDEYpvVQAqDY1WnFVpqJYXOKyWYySgbDfiPAyjtcW1FJNxhkN6tqI6r4vG6Ags8Mow1sg9RxWq4_2WRhSyQUf9DUMeg-2OBxY7npnTMNHLxpiVmg4QAA)**
an invalid `mpr`:

`INSERT INTO Contents VALUES (100,'example','2000-2-2','R-18',5,100);`

**![](https://lh3.googleusercontent.com/vaukP4vh0kGjYubTY8MxHOJ6OXulE7Uh2WQaZ3b2dF1M0h75JddpDe0e7Owx5bW4jI1ha5jQXNS30JazbNjhR3GSUXkm3OKTkBS377BRi6aaAy9_X1kl5NVYXQb-yfhuAC7FJncL4zszkEfvIkI_LMDDOUJkAmXh6fikwKBEVgmw5FMX3NckxKPg90WzJA)**

an invalid `imdb_rating`:

`INSERT INTO Contents VALUES (100,'example','2000-2-2','R',-5,100);`

**![](https://lh3.googleusercontent.com/bX2PQIUQQJcfzWwm9WqXh1QbrVFDF38QZg6jR4LuaX5PMeHQ1uIRk9C-bV_FG-ttjzEkGAxUr6_8Xbjw9x26hgqZOv88h0EVFURqvujd1Rjp764riOYIQdgNmjR3uqs0ljpffbv4-7-7VePBJzjyEDY-9HvxI3FhUEiqwmua5o2cM64jm3Tb7iak9aVMMA)**

an invalid `num_of_reviews`:

`INSERT INTO Contents VALUES (100,'hi','2000-2-2','R',5,-9);`

**![](https://lh6.googleusercontent.com/VVSYJYYNOAwl421Eq1jileF__k2kmGKvlU4lV8x9d6yo8tlaIjmu4ElIBXt7uxd5Gt8oEQ_n_TwmS8LejTEi1XHiQwHQtQgSy8IG2kvCWHKWcS5OMKxsr9g1CPzoUP0h7rs7tOvwilqCuTdyCdW2nqaPoG7K2fOiDixAhF10GWOgnjbONEdGMNwlijgGBQ)**