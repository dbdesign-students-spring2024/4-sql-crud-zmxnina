# SQL CRUD
## Part 1: Restaurant Finder
### Table Structure
Create the required tables ```restaurants``` and ```reviews```:
```
DROP TABLE IF EXISTS restaurants;
CREATE TABLE restaurants (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    category TEXT,
    price_tier TEXT,
    neighborhood TEXT,
    opening_hours TEXT,
    average_rating REAL,
    good_for_kids INTEGER NOT NULL -- 0 for false, 1 for true
);

DROP TABLE IF EXISTS reviews;
CREATE TABLE reviews (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    restaurant_id INTEGER NOT NULL,
    rating REAL CHECK (rating >= 1 AND rating <= 5),
    review TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(id)
);
```
### Practice Data
Link to the practice CSV data file:
- [restaurants.csv](data/restaurants.csv)

Import the practice CSV data file into the ```restaurants``` table:
```
.mode csv
.headers on
.import data/restaurants.csv restaurants
```

### Queries
1. Find all cheap restaurants in Soho.
    ```
    SELECT * FROM restaurants WHERE neighborhood = 'Soho' AND price_tier = 'Cheap';
    ```

2. Find all Japanese restaurants with 3 stars or more, ordered by the number of stars in descending order.
    ```
    SELECT * FROM restaurants WHERE category = 'Japanese' AND average_rating >= 3 ORDER BY average_rating DESC;
    ```

3. Find all restaurants that are open now.
    ```
    SELECT * FROM restaurants WHERE SUBSTR(opening_hours, 1, 5) <= strftime('%H:%M', 'now', 'localtime') AND SUBSTR(opening_hours, 7, 5) >= strftime('%H:%M', 'now', 'localtime');
    ```

4. Leave a review for restaurant 77.
    ```
    INSERT INTO reviews (restaurant_id, rating, review, created_at) VALUES (77, 4.92, 'Authentic Hunan cuisine, will visit again!', DATETIME('now', 'localtime'));
    ```

5. Delete all restaurants that are not good for kids.
    ```
    DELETE FROM restaurants WHERE good_for_kids = 0;
    ```

6. Find the number of restaurants in each NYC neighborhood.
    ```
    SELECT neighborhood, COUNT(id) AS number_of_restaurants FROM restaurants GROUP BY neighborhood;
    ```

## Part 2: Social Media App
### Table Structure
Create the required tables ```users``` and ```posts```:
```
DROP TABLE IF EXISTS users;
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT NOT NULL UNIQUE,
    password TEXT NOT NULL,
    handle TEXT NOT NULL
);

DROP TABLE IF EXISTS posts;
CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER,
    type TEXT NOT NULL, 
    content TEXT NOT NULL,
    recipient_id INTEGER, -- NULL for stories
    viewed_by_recipient INTEGER, -- 1 for viewed, 0 for not viewed, NULL for stories
    visible INTEGER DEFAULT 1, -- 1 for visible, 0 for invisible 
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (recipient_id) REFERENCES users(id)
);
```

### Practice Data
Links to the practice CSV data files:
- [users.csv](data/users.csv)
- [posts.csv](data/posts.csv)

Import the practice CSV data files into their respective tables:
```
.mode csv
.headers on
.import data/users.csv users
.import data/posts.csv posts
```

### Queries
The dummy data within the practice CSV file is configured such that only stories created within the 24-hour timeframe from 2024-02-28 00:00:00 to 2024-02-29 00:00:00 are visible to the users. Considering that the grader may be viewing this at a later date, we can first update all stories that were created more than 24 hours before the current viewing time to be invisible. This ensures that the succeeding queries accurately select only the most recent stories that are still visible within the current 24-hour window.
```
UPDATE posts SET visible = 0 WHERE type = 'Story' AND ROUND((JULIANDAY('now', 'localtime') - JULIANDAY(created_at)) * 24) > 24;
```

1. Register a new user.
    ```
    INSERT INTO users (email, password, handle) VALUES ('user123@example.com', 'password123!', 'user123');
    ```

2. Create a new message sent by user 9 to user 99.
    ```
    INSERT INTO posts (user_id, type, content, recipient_id, viewed_by_recipient, visible, created_at) VALUES (9, 'Message', 'Hello, how are you?', 99, 0, 1, DATETIME('now', 'localtime'));
    ```

3. Create a new story by user 9.
    ```
    INSERT INTO posts (user_id, type, content, visible, created_at) VALUES (9, 'Story', 'Check out my new story!', 1, DATETIME('now', 'localtime'));
    ```
4. Show the 10 most recent visible messages and stories, in order of recency.
    ```
    SELECT * FROM posts WHERE visible = 1 ORDER BY created_at DESC LIMIT 10;
    ```

5. Show the 10 most recent visible messages sent by user 9 to user 99, in order of recency.
    ```
    SELECT * FROM posts WHERE user_id = 9 AND recipient_id = 99 AND type = 'Message' AND visible = 1 ORDER BY created_at DESC LIMIT 10;
    ```

6. Make all stories that are more than 24 hours old invisible.
    ```
    UPDATE posts SET visible = 0 WHERE type = 'Story' AND ROUND((JULIANDAY('now', 'localtime') - JULIANDAY(created_at)) * 24) > 24;
    ```

7. Show all invisible messages and stories, in order of recency.
    ```
    SELECT * FROM posts WHERE visible = 0 ORDER BY created_at DESC;
    ```

8. Show the number of posts by each user.
    ```
    SELECT user_id, COUNT(id) as number_of_posts FROM posts GROUP BY user_id;
    ```

9. Show the post text and email address of all posts and the user who made them within the last 24 hours.
    ```
    SELECT posts.content, users.email FROM posts INNER JOIN users ON posts.user_id = users.id WHERE ROUND((JULIANDAY('now', 'localtime') - JULIANDAY(posts.created_at)) * 24) <= 24;
    ```

10. Show the email addresses of all users who have not posted anything yet.
    ```
    SELECT users.email FROM users LEFT JOIN posts ON users.id = posts.user_id WHERE posts.user_id IS NULL;
    ```