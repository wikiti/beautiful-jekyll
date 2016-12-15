---
layout: post
published: false
title: 'Rails: Download link counter'
---
It's 4:00 AM in the morning. You're finishing your amazing Rails WebApp which will let final users download free content without paying anything. You're almost done, polishing some details. However, you lack of one feature, and you don't know how to implement it (*really? are you serious? after all this?*): track how many times has a file been downloaded.

Fear not, my friend! I've the solution for you, right on this post.

## Problem

As stated on that amazing introduction, we need to *track* how many times a file has been downloaded. There is no way for Rails routes to persist how many times a route has been accesed (actually, it can, but rack middleware is required). So, how do we solve this?

## Solution

Models. Yup. That's it. We'll store how many times a link has been accessed by using a database record. We need to store the path (e.g. the file route) and the download count. There are many ways to store that information in the database. For example:

- Store `download_count` into an existent model.
- Create a new model, storing `controller`, `action`, `file_id` ... and `download_count`.
- Create a new model, storing the `path` and `download_count`.

I think that the third option is the most flexible of all of them, becase we can track anything we want; from file download to per route access, building some cool stats for ~~software destroyers~~ marking team.

Now, back to work!

## Implementation

## Conclussions
