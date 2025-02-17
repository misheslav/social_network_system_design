# Social Network for travelers

## Functional requirements

<ul>
    <li>Publishing travel posts with photos, short descriptions and link to a specific place.</li>
    <li>Rating and comments on posts of other travelers.</li>
    <li>Subscription on other travelers to monitor their activity via news feed.</li>
    <li>User should be able to make a context search among the places by providing its title as a pattern.</li>
    <li>Viewing other traveler's feeds and a user's feed based on subscriptions in reverse chronological order.</li>
</ul>

## Non-functional requirements

<ul>
    <li>10 000 000 DAU.</li>
    <li>Availability of four nines.</li>
    <li>The business is currently aimed only at the audience of CIS countries.</li>
    <li>High demand is expected during the holiday season (summer months) and long holidays (New Year's holidays).</li>
    <li>User make a post once a week, we are expecting this number to be up to 3 during the season time. 
        One post can contain up to 10 photos (1280 * 720) and 1k of UTF-8 characters for description, 500 UTF-8 characters per comment.</li>
    <li>The average user has 50 subscriptions and views the feed 5 times a day.</li>
    <li>The average user make a search for a spot 3 times a day during the season</li>
    <li>Data should be 100% reliable. If user make a post, system will guarantee that it will never be lost.</li>
</ul>

## Basic calculations

### Media assumptions
> One photo size = 1280 * 720 * 3b(bit depth) ~= 2.8 MB
> It's assumed that when user list a feed, we are showing him one photo from the post as preview
> When user reads a post we are showing him photos in carousel format, meaning he can see only one photo at a time.

### Feed assumptions

> One user make 3 posts per week, and has 50 subscriptions, so average daily feed size will be\
> 50 * 3(posts per week) / 7 ~= 21 post\
> It's proposed to break the result into pages (for example 5 posts per page)

### Posts

```
Assuming that each user creates 3 post per week
RPS(write) = 10 000 000 * 3 / 86400 * 7 ~= 50
Attaching photos: RPS(write) = 10 000 000 * 3 * 10 / 86400 * 7 ~= 496
RPS(read) = 10 000 000 * 5 / 86400 ~= 579
Downloading photos: RPS(read) = 10 000 000 * 5(times per day) * 5(posts per page) / 86400 ~= 2893
```

### Comments

```
Assuming that each user make 5 comments per week, and read 10 comment per post
RPS(write) = 10 000 000 * 5 / 86400 * 7 ~= 83
RPS(read) = 10 000 000 * 1(we can paginate comments. 10 per page) / 86400 ~= 115
```

### Ratings
```
One user make 3 posts per week, and has 50 subscriptions, so average daily feed size will be:
50 * 3 / 7 ~= 21 post
Let`s assume that user will rate a half of his feed, so he will do 10 requests
RPS(write) = 10 000 000 * 10 / 86400 = 1157
```

### Spots
```
Lets assume that we need to show top 3 most rated posts per spot
RPS(read) = 10 000 000 * 3 / 86400 ~= 347
Downloading photos: RPS(read) = 10 000 000 * 3(times per day) * 3(posts) / 86400 ~= 1041
```

## Traffic estimations

### Post structure
```
    - id - post id, long
    - userId - user id, long
    - createdAt - instant, long
    - text - post description, UTF-8
    - comments - number of comments for post, int
    - rates - number of rates, int
    - files - array of photo links, 1 link = 50 ascii symbols
    - position (lat double, lon double)

Post size = 8bytes + 8bytes + 8bytes + 4kB(1000 * 4byte) + 4byte + 4byte + 10 * 50bytes + 8bytes(lat) + 8bytes(lon) ~= 4.6kB
```

### Comments structure
```
    - id, comment id, long
    - userId, user id, long
    - text, up to 500 chars of UTF-8,
    - createdAt, instant, long
Comment size = 8bytes + 8bytes + 2kB(500 * 4byte) + 8bytes ~= 2kB
```

### Rate structure
```
    - userId, user id, long
Rate size = 8byes
``` 

### Traffic for post
```
    Making new post:
        Attaching media: 496 * 2.8MB ~= 1.4GB/sec
        Attaching metadata: 50*4.6kB ~= 230kB/sec
    Reading feed:
        Reading metadata: 579 * 4.6kB = 2.6Mb/sec
        Downloading photo per post: 2893 * 2.8MB = 8.1GB/sec
    Adding a comment: 83 * 2kB = 166 kB/sec
    Reading a comments: 115 * 2Kb = 230 kB/sec
    Rating a post: 1157 * 8b = 9kB/sec
```

### Traffic for spots
```
    Reading metadata: 347 * 4.6kB = 1.6Mb/sec
    Downloading photo per post: 1041 * 2.8MB = 2.9Gb/sec
```
