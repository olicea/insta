2. Requirements and Goals of the System#

We’ll focus on the following set of requirements while designing Instagram:

Functional Requirements

    Users should be able to upload/download/view photos.
    Users can perform searches based on photo/video titles.
    Users can follow other users.
    The system should generate and display a user’s News Feed consisting of top photos from all the people the user follows.

Non-functional Requirements

    Our service needs to be highly available.
    The acceptable latency of the system is 200ms for News Feed generation.
    Consistency can take a hit (in the interest of availability) if a user doesn’t see a photo for a while; it should be fine.
    The system should be highly reliable; any uploaded photo or video should never be lost.

Not in scope: Adding tags to photos, searching photos on tags, commenting on photos, tagging users to photos, who to follow, etc.



# Naive design considerations

no limits on upload frequency
no limits on download frequency
infinite retention


# Space considerations

Assume 100M users, 1M daily users
2M new posts (2 per day per user)

Average photo size 4MB

4MB * 2M = 8GB per day * 100M = 800GB per year

# High level design

2 main scenarios:
* upload photos
* review feed

workflows
* register new user
* follow
* unfollow


# system design


User | ----->  NLB |    web servers  ----->  database image metadata, user metadata
/post |                              ----->  database image data (immutable blobs)
/feed | 



# API Design

Contracts

User:
    UserId: long
    Name: string
    Email: string

Subscriptions
    SubscriptionId: long
    FollowerId: long
    FollowingId: long

Post:
    PostId: long
    CreatedTime: datetime
    UserId: long
    Title: string
    Content: binary
    ContentType: int //could be: text, image, video


# Database Design

Relational database, due to the relationship between users and users they follow


{user} 1 to many {follows}
{user} 1 to many {posts}


posting an image

image_id = upload image into blob storage (id) 


insert 
into    posts
set     userId = userid
        tilte = title
        imageId = image_id
        ...
        createdtime = now()



querying feed
    select all posts
    from all users this user follows
    from this user
    order by time 
    take number_od_posts_per_page
    skip number_of_posts_per_page * page_number


index users by id
index posts by id, created time
index follows by userid


# Web Api

    POST user/register
    registers a new user 
    {
        "userId" : 1 
        "name" : "John Doe"
        "email" : "john.doe.com"
    }

    response example
    {
        "userId" : 1 
        "name" : "John Doe"
        "email" : "john.doe.com"
    }

 * POST post/create
    creates a post for a specific user

    {
        "userId" : 1 
        "title" : "My first post"
        "content" : "This is my first post"
        "contentType" : "text"
    }

    response HTTP 201
    {
        "postId" : 1
        "createdTime": "2022-0101T00:00:00"
        "userId" : 1 
        "title" : "My first post"
        "contentType" : "text"
        "content" : 
    }

    HTTP 400 when a bad user is used
    HTTP 400 when a long title, or long description is used
    HTTP 503 when the number of posts per minute are exceeded

* DELETE POST post/delete/userId}/{postId}

    returned HTTP 204 No content success!
    HTTP 404 when the {userId} or {postId} or user does not exit 


* FOLLOW POST user/follow{userId}/{userId}

    HTTP 201 Created
    {
        "subscriptionId" : 1
        "followerId" : 1
        "followingId" : 2
    }


* UNFOLLOW DELETE user/follow{userId}/{userId}

    HTTP 204 No Content 
    {
        "subscriptionId" : 1
        "followerId" : 1
        "followingId" : 2
    }


* GET feed/{userId}

    returns a list of posts for the user, as well as a list of posts from users they follow, order by date on a descending order


    HTTP 404 when the {userId} does not exit



# Availability


* Separate web api from storage api
* Separate upload and download operations (microservices)
* Multiple instances of upload and download services



Separate storage 
* shard users into several databases, naive:
    keep a set number of shards (10)
     shard per user id hash,   => userID % 10
     pros: isolation
     cons: hard to scale to 10+ sicne hte shard algoryth is  % {number_of_hards} 
* shard images into several accounts/physical disks, naive: shard 


---


scale

load balancing
hot users (caching) 


