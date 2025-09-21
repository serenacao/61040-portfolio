# Pset 2

## Concept Questions
1. Contexts. The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

    ```
    Contexts seem to be a sort of namespace with which the link prefixes exist inside of, that identify the scope with which nonces must be unique within. For a URL shortening app, it could be a user or an entire service.
    ```

2. Storing used strings. Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

    ```
    This maintains an invariant that generate cannot return a nonce that has already been used. If we had a counter, since the counter will increase with every call to generate, we will always have a unique nonce, and we can use the generated counter number as our nonce (0, 1, 2, ...)
    ```


3. Words as nonces. One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?

    ```
    One advantage of this is comprehensibility and it is easier to remember as opposed to a string of random letters. However, there are less available words as opposed to random string generation, so if the user was very ambitious with making links it could be seen as a disadvantage. To tweak our original concept, I would add a  dictionary of available "Words" that is selected from, and modify generate to select from this dictionary.
    ```

## Synchronization Questions

1. Partial matching. In the first sync (called generate), the Request.shortenUrl action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called register) both appear. Why is this?

    ```
    The targetUrl is irrelevant to the actual generation of the nonce, but registration needs the targetUrl since it is registering the shortUrl to link to the targetUrl.
    ```

2. Omitting names. The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?

    ```
    We omit the names where we know the inputs that are being passed in. However, if it is ambiguous (it depends on the output of another action), we need to explicitly denote where we get the input from.
    ```

3. Inclusion of request. Why is the request action included in the first two syncs but not the third one?

    ```
    Setting expiry assumes that the link has already been registered, so it is not necessary to include the process of link creation. The first two syncs need request since the client is initiating with inputs. 
    ```


4. Fixed domain. Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?

    ```
    I would implement it by hard coding the context to be bit.ly via some global constant that I would pass in, in place of the original context.

    sync generate
    when Request.shortenUrl (targetUrl)
    then NonceGeneration.generate (context: FIXED_DOMAIN)

    sync register
    when
        Request.shortenUrl (targetUrl)
        NonceGeneration.generate (): (nonce)
    then UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase: FIXED_DOMAIN, targetUrl)

    sync setExpiry
    when UrlShortening.register (): (shortUrl)
    then ExpiringResource.setExpiry (resource: shortUrl, seconds)
    ```

5. Adding a sync. These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the ExpiringResource and URLShortening concepts.

    ```
    sync expireResource 
    when ExpiringResource.system expireResource(): (resource)

    then UrlShortening.delete(shortUrl: resource)
    ```

## Extending Questions
1. Design a couple of additional concepts to realize this extension, and write them out in full (but including only the essential actions and state). It should not be necessary to make any changes to the existing concepts.

    ```
    concept UrlAnalytics

    purpose user analytics on short links, including count accesses 

    principle after a short link is created by the user, the user can choose to see analytic information about it, including count access

    state

    a set of Counts
        a ShortUrl shortUrl 
        an Owner string
        an AccessCount number


    actions 

    track(shortUrl: ShortUrl):()
        requires: shortUrl does not exist already
        effects: adds shortUrl to set with accessCount 0 

    increment(shortUrl: ShortUrl): (shortUrl: ShortUrl)
        requires: shortUrl exists
        effects: increments shortUrl accessCount by 1
    
    getAccessCount(shortUrl: ShortUrl): (AccessCount: number)
        requires: shortUrl exists

    delete(shortUrl: ShortUrl): ()
        requires: shortUrl exists
        effects: deletes shortUrl from set

    ```

    ```
    concept Ownership

    purpose keeps track of url creators 

    principle after a shortUrl is created by a user, the user owns it

    state
        a set of Ownership
            a ShortUrl shortUrl
            an Owner string

    actions
    assign (user: User, shortUrl: ShortUrl): (ownership: Ownership)  
        requires: shortUrl does not already exist in set

    getUser(shortUrl: ShortUrl): (owner: Owner)
        requires: shortUrl exists 
    ```

2. Specify three essential synchronizations with your new concepts: one that happens when shortenings are created; one when shortenings are translated to targets; and one when a user examines analytics.
    ```
    sync register
    when
        Request.shortenUrl (targetUrl, shortUrlBase)
        NonceGeneration.generate (): (nonce)
    then UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase, targetUrl): (shortUrl)

    sync trackAccess
    when 
        UrlShortening.register(): (shortUrl)
    then
        CountAccess.track(shortUrl)
        Ownership.assign(user, shortUrl)
    
    
    sync clickAccess
    when
        UrlShortening.lookup(shortUrl):(targetUrl)
    then
        CountAccess.increment(shortUrl)

    sync expireAccess
    when 
        ExpiringResource.system expireResource (): (resource)
    then 
        CountAccess.delete (shortUrl: resource)
    
    sync viewAnalytics
    when 
        Request.viewAnalytics(user, shortUrl)
        Ownership.getUser(shortUrl): (owner)
        requires owner = user
        CountAccess.getAccessCount(shortUrl): (count)
    then 
        Response.returnAnalytics (count)
    ```

3. As a way to assess the modularity of your solution, consider each of the following feature requests, to be included along with analytics. For each one, outline how it might be realized (eg, by changing or adding a concept or a sync), or argue that the feature would be undesirable and should not be included:

- Allowing users to choose their own short URLs;

    ```
    Seems difficult to implement, this might allow users to do something like www.google.com or 'cats' and we would have to check for correct formatting, this is more similar to a hyperlink in a document. I think we would need to add a url policy concept that limits what users can choose. I think it would be possible, but would have to be done carefully.
    ```

- Using the “word as nonce” strategy to generate more memorable short URLs;

    ```
    I think this is a nice concept, and I think it's actually quite convenient if you are making a link in a class to something with no qr scanning code (yellkey). Already was implemented above, and would not affect our new concept, since our new concept does not depend on form of URL. Only modify generation of nonces.
    ```

- Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL;

    ```
    We would have to keep track of a count for the targetUrls as well, and we would have to adjust the state of our original concept to also register the target url whenever a short url is registered. Would only be changing the URL analytics concept
    ```

- Generate short URLs that are not easily guessed;
    ```
    I think this is pretty subjective, but I think this could be done via a long string of generated numbers and letters and other characters, which will form the nonce, which would be through changing the url generation concept
    ```
- Supporting reporting of analytics to creators of short URLs who have not registered as user.

    ```
    This does not sound possible, as we need a way to verify that it is the creator of the url, which can only be done through a registration + login.
    ```
