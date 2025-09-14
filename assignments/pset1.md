# Problem Set 1: Reading and Writing Concepts

## Exercise 1: Reading a concept

1. What are two invariants of the state? (Hint: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves 
it.

    A purchase can only be fulfilled if the request count is at least the purchase count

    Cannot have multiple requests counts for same item -- must be one count

    I think that the first invariant is more important, since otherwise, users could overpurchase gifts for a person (if the person requests 5 gifts, we could buy them 10).

2. Fixing an action. Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

    Count is not required to be positive, hence we could technically call purchase (item: Item, count: -1), and if item is at 0, after this action (since count < number), we have item at 1. We could fix it by requiring count to always be non-negative.


3. Inferring behavior. The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

    Yes, it could be. It might be to restock, or to remove an Item before a user purchases it. Either way, closing the registry allows the user who owns it to change it without worrying about purchase interactions. 

4. Registry deletion. There is no action to delete a registry. Would this matter in practice?

    No, we could honestly just reuse the same registry over and over without creating a new one, since we have the option to deleteItem, addItem.

5. Queries. What are two common queries likely to be executed against the concept state? (Hint: one is executed by a registry owner, and one by a giver of a gift.)

    “list items in registry” for the owner, since the owner would want to see what items have been added so far, and “list available items” for the giver, since the giver wants to see what is available to give. 


6. Hiding purchases. A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?

    I would include a hidden flag. I think that the hiding aspect would be handled by another concept (like the page layout, or how it shows the purchases to the user) and we could use the hidden flag to determine if the user can see the purchases or not.


7. Generic types. The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

    Because this is a concept that should be separate from the concept of gift giving, we want to allow for separation of concerns/modularity.  

## Exercise 2: Writing a concept

1. Complete the definition of the concept state.

**concept** PasswordAuthentication

  **purpose** limit access to known users

  **principle** after a user registers with a username and a password,
    they can authenticate with that same username and password
    and be treated each time as the same user

  **state**

    a set of Users
        a username String 
        a password String

  **actions**

    register (username: String, password: String): (user: User)
        requires: username does not exist already
        effect: creates a new User with username and password

    authenticate (username: String, password: String): (user: User)
        requires: user with username and password to exist
        effect: returns user


    
2. Write a requires/effects specification for each of the two actions. (Hints: The register action creates and returns a new user. The authenticate action is primarily a guard, and doesn’t mutate the state.)

Completed above

3. What essential invariant must hold on the state? How is it preserved?

Usernames must be unique across different users. It is preserved via the register function, which is the only mutator of our users set.

4. One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality. (Hints: you should add (1) an extra result variable to the register action that returns a secret token that (via a sync) will be emailed to the user; (2) a new confirm action that takes a username and a secret token and completes the registration; (3) whatever additional state is needed to support this behavior.)

**concept** PasswordAuthentication
  purpose limit access to known users
  principle after a user registers with a username and a password,
    they can authenticate with that same username and password
    and be treated each time as the same user

  **state**

    a set of Users
        a username String 
        a password String
        a token String
        a email String
        a confirmed flag

    

  **actions**

    register (username: String, password: String, email: String): (user: User, token: String)
        requires: username does not exist already
        effect: creates a new User with username, password, false confirmed, and a special token

    confirm(username: String, token: String): (user: User)
        requires: USER with username, token , false confirmed flag exists 
        effect: sets confirmed flag to true

    authenticate (username: String, password: String): (user: User)
        requires: user with username and password to exist
        effect: returns user

## Exercise 3: Comparing concepts

 A concept specification for PersonalAccessToken and a succinct note about how it differs from PasswordAuthentication and how you might change the GitHub documentation to explain this.

**concept** PersonalAccessToken

**purpose** Allows for external access to some user's resources with a certain set of permissions without needing to expose additional information

**principle** after a user creates a PAT with a certain set of permissions, any external user can authenticate with that PAT and be treated as the user within the set scope and permissions

 **state** 

    a PAT
        a User
        a set of Permissions

**actions**

    createPAT(user: User, permissions: Permissions): (pat: PAT)
    requires: existing user
    effect: creates a pat for user with permissions

    authenticate(user: User, pat: PAT): (pat:PAT)
    requires: existing user, existing PAT
    effect: return the PAT if the PAT exists under the user

**Main difference:**

    PATs contain a limited scope/permissions and are revocable, while a username + password allows for full access and cannot be revoked. Generally, the login allows for direct access within Github while PATs are better for CI/CD pipelines/external workflows. GitHub docs could emphasize “PATs are designed for tools and automation, with adjustable permissions and revocability” (the purpose of a PAT) instead of focusing on the lack of passwords

## Exercise 4: Defining familiar Concepts

URL Shortener

Notes: suffix cannot be repeated, otherwise this will lead to link collisions

**concept** URLShortener(User)

**purpose** makes links easier to share, more accessible, and readable

**principle** after a user enters a URL to some website and optional parameters (like suffix), they can access the website with a shorter URL

**state**

    a set of TinyURL
        a user User
        a url String
        a suffix String


**actions**

    createUrl(user: User, url: String, suffix: String): (tinyURL: TinyURL)
    requires: suffix is unique across all suffixes, or suffix is None
    effect: creates tinyURL with suffix if suffix exists, otherwise with a randomly generated suffix, under user that redirects to url

    deleteUrl(user: User, tinyURL: String): (tinyURL: TinyURL)
    requires: tinyURL exists and user is owner of it
    effect: deletes tinyURL 

---

**Conference Room Booking**

Note: Overlapping slots is not allowed, and only one booking is allowed per slot

**concept** RoomBooking(User)

**purpose** allows users to book rooms in advance

**principle** after a user requests an available time slot for a specific room, the user has it for that entire slot

**state**

    a set of Slots
        a roomId
        a date Date
        a startTime Time
        a endTime Time

    a set of Bookings
        a slot Slot
        a user User
        a roomId String

**actions**

    createSlot(date: Date, startTime: Time,endTime: Time, roomId: String): (slot:Slot)
        requires: roomId exists, endTime after startTime, startTime to endTime has no pre-existing overlapping slots on date 
        effect: creates new slot with start and end times given for roomId

    book(user: User, slot: Slot): (booking: Booking)
        requires: slot exists and is not already booked
        effect: creates booking for slot under user 

    cancel(user: User, slot: Slot):(booking: Booking)
        requires: slot exists under a user that exists
        effect: deletes booking for slot under user

---

**Electronic Boarding Pass**

Note: Cannot have duplicate seatIds for same flightDetails, tokens ensure that the boardingPass belongs to the airline system

**concept** BoardingPass(FlightDetails, Device)

**purpose** provide a verifiable electronic credential that allows a user to board their flight and stay updated on information regarding their flight 

**principle** after a user adds their boarding pass to their phone, it updates them on their flight gate, boarding time, departure, and can also be used to go through security board 

**state**

    a set of BoardingPasses, each with:  
        a passenger String
        a flightDetails FlightDetails
        a seatId String 
        a token String
        a boarded flag
        
**actions**

    createPass(passenger: String, flightDetails: FlightDetails, a seatId: String): (boardingPass: BoardingPass)
        requires: seatId does not exist
        effect: creates boardingPass for passenger on flight flightDetails and seat seatId

    addPass(boardingPass: BoardingPass, device: Device): (boardingPass: BoardingPass)
        requires: boardingPass exists
        effect: stores pass on device

    updatePass(boardingPass: BoardingPass, flightDetails: FlightDetails):(boardingPass: BoardingPass)
        requires: boardingPass exists
        effect: updates boardingPass with new flightDetails

    board(boardingPass: BoardingPass):(boardingPass: BoardingPass)
        requires: boardingPass exists and the token matches the token in the airline database
        effect: updates boarded flag to be True


