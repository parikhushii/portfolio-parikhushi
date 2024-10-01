---
title: A3
layout: doc
---

# A3: Convergent Design

## Pitch

**Short Pitch**

APPNAME is a service dedicated to transitioning online interactions offline, targeted towards recent graduates starting a new chapter of their lives in an unfamiliar social climate. This user base stands to benefit the most from this service because they have a wealth of online connections but tend to introversion as a response to living in the “real world” for the first time. 

In my A1 interviews, I grappled with a peculiar dichotomy: social media has the capacity to meaningfully connect one subset of users (Jay and his ex-girlfriend, ex-friend, and coworker) but superficially connect and isolate another subset (Bryson and subscribers borrowed from the family vlog channel). APPNAME reframes social media as a means to a deeper, offline relationship instead of an end in itself – to increase account engagement or follower count. 

Like a catalyst in a chemical reaction, APPNAME lowers the activation energy, or inhibitions, associated with inviting an online connection to interact offline. Every logistic detail can become a source of anxiety that dissuades a person from proposing an invitation: “Will they like what I have planned?”, “Will they be free?”, “Will they accept?” APPNAME resolves these worries. Users can **label recommended** activities as part of a Favorite or Wishlist collection, **sync** with other calendars to **label** blocked and free time and send a **request** to an online connection to meet at their Favorite place during their free time. 

![image](/../assets/images/A3/ActivationEnergy.jpg)

**Detailed Pitch**

Two observations from my needfinding interviews guide this MVP: 
1. Social media has the capacity to connect people meaningfully, but the final form of such a connection is offline interaction. Transitioning an online connection offline frames interactions on social media as the *means* to a deeper relationship, instead of the *end* itself – to increase engagement levels or follower count. 
2. For many people, inviting an online connection to interact offline is *hard.* Every logistic detail becomes a source of anxiety that dissuades them from proposing an invitation: “Will they like what I have planned?”, “Will they be free?”, “Will they accept?”  

APPNAME catalyzes a transition from an online to offline connection by lowering the inhibitions associated with inviting: 
+ Concepts within the app resolve logistic doubts by revealing information about each user’s preferences and availability. 
+ Culturally, we interpret requests that are standardized and sent on a social media platform (e.g., a follow request on Instagram or a Like on Hinge) as less imposing than ones that are personalized and proposed via text or in-person. I invoke this behavioral pattern to simultaneously relieve pressure off the invite’s sender and recipient. 

APPNAME offloads the responsibility of connecting users to other social media apps, which has two consequences: 
1. APPNAME is modular. Users can continue to connect on their favorite platforms and transition offline with APPNAME regardless of how they make a connection. 
2. By crystallizing the possibility of meeting offline, APPNAME has the potential to shift how we use other social media from consuming to connecting. 

## Functional Design

### Collapsing Features into Concepts
**Purpose Statement:** *This app would catalyze a transition from an online to offline connection by lowering the inhibitions associated with inviting.* 

While grouping features into concepts and selecting a subset of the resulting concepts to include in the app MVP, I evaluate the feature or concept against the purpose statement: 
+ How well does this feature or concept fulfill the app’s purpose?  
+ Is there another feature or concept that does the same thing? 
+ Is a feature central enough to the app’s purpose to build into a self-contained concept? 

This diagram demonstrates purpose-centric convergence of features to concepts to the subset to include in the app MVP: 
![image](/../assets/images/A3/AlignmentHeatmap.jpg)

### Concepts

1. **Authenticating**
    ~~~
    concept Authenticating[User]

    purpose
    authenticate users so that app users correspond to people

    operational principle
    after a user u registers with a username n and password p, they can authenticate as user u by providing the pair of n and p

    state
    credentials: set User
    username: User -> one String
    password: credentials -> one String

    register(u: User, n: String, p: String)
        if n ∉ {v.username | v ∈ credentials} then
            credentials := credentials ∪ {u}
            u.username := n
            u.password := p

    authenticate(u: User, n: String, p: String)
        if ∃ u (u.username = n AND u.password = p) then
            return True
        else
            return False

    delete(u: User)
        if u ∈ credentials then
            credentials := credentials - {u}
            u.username := none
            u.password := none

    ~~~
2. **Sessioning**
    ~~~
    concept Sessioning[User]
    
    purpose
    enable authenticated actions for an extended period of time

    operational principle
    after a session s starts for user u, and as long as it is active, we can identify u as associated with s

    state
    active: set Session
    user: Session -> one User

    actions:
    start(u: User)
    s := new Session()
    s.user := u
    active := active ∪ {s}

    getUser(s: Session, OUT u: User)
    return s.u

    end(s: Session)
    active := active - {s}
    ~~~
3. **Expiring**
    ~~~
    concept Expiring[Resource]

    purpose
    handle expiration of short-lives resources

    operational principle
    if you allocate a resource r for t seconds, after t seconds, the resource expires

    state
    active: set Resource
    expiry: Resource -> one Date
    
    actions
    allocate(r: Resource, t: Date)
    if r ∉ active then
            active := active ∪ {r}
    r.expiry = t

    expire(r: Resource)
    if r.expiry is before now then
        active := active - {r}  
        r.expiry := none

4. **Labeling**
    ~~~
    concept Labeling[Element]

    purpose 
    assign elements to overlapping or disjoint groups

    principle 
    if element e has label l, then e is a member of group l

    state
    labels: set Label
    elements: Label -> set Element
    disjoint: Label -> one Boolean

    actions
    addLabel(l: Label, e: Element)
        if l ∉ labels then
            labels := labels ∪ {l}
        if l.disjoint then
            l.elements := {e}
            for each label l' ∈ labels where l' ≠ l do
                l'.elements := l'.elements - {e}
        else
            l.elements := l.elements ∪ {e}

    removeLabel(l: Label, e: Element)
        l.elements := l.elements - {e}

    lookup(l: Label) -> set Element
        return l.elements
    
    ~~~
5. **Requesting**
    ~~~
    concept: Labeling[Request]

    purpose 
    communicate a request from sender to recipient(s)

    principle 
    if sender s sends a request r with optional capacity c to a queue of recipients q, the first c recipients in q can accept or decline r, and r is sent down q until c is full or q is finished

    state
    requests: set Request
    accepted: Request -> set User
    pending: Request -> set User
    queue: Request -> list User
    capacity: Request -> opt Integer

    actions
    send(r: Request)
        requests := requests ∪ {r}
        let initialRecipients = queue[r][0 : r.capacity]
        queue[r] := queue[r] - initialRecipients
        pending[r] := pending[r] ∪ initialRecipients
    
    accept(r: Request, recipient: User)
        r.pending := r.pending - {recipient}
        r.accepted := r.accepted ∪ {recipient}
        if |r.accepted| = r.capacity then
            requests := requests - {r}

    decline(r: Request, recipient: User)
        r.pending := r.pending - {recipient}
        if |r.queue| = 0 then
            requests := requests - {r}
        else
            let nextRecipient = r.queue[0]
            r.pending := r.pending ∪ {nextRecipient}

    cancel(r: Request, recipient: User)
        r.accepted := r.accepted - {recipient}
    ~~~
6. **Syncing**
    ~~~
    Syncing[Data]

    purpose 
    ensure data across multiple sources remains consistent and up-to-date

    principle 
    if data d in source s is modified, then the change is propagated to targets t

    state
    datasets: set Dataset
    isSource: Dataset -> Boolean
    data: Dataset -> set Data

    actions

    pull(datasets: set Dataset)
        let sources = { ds ∈ datasets | ds.isSource = true }
        let targets = { ds ∈ datasets | ds.isSource = false }
        
        for each source s ∈ sources do
            for each d ∈ s.data do
                if d ∉ t.data for each target t ∈ targets then
                    t.data := t.data ∪ {d}

    push(datasets: set Dataset)
        let sources = { ds ∈ datasets | ds.isSource = true }
        let targets = { ds ∈ datasets | ds.isSource = false }
        
        for each target t ∈ targets do
            for each d ∈ t.data do
                if d ∉ s.data for each source s ∈ sources then
                    s.data := s.data ∪ {d}

    addSource(datasets: set Dataset, s: Dataset)
        s.isSource := True
        datasets := datasets ∪ {s}

    addTarget(datasets: set Dataset, t: Dataset)
        t.isSource := False
        datasets := datasets ∪ {t}
    ~~~
7. **Recommending**
    ~~~
    concept Recommending[Element]

    purpose 
    show users the most preferable elements first

    principle 
    if recommendations are enabled, user u sees a reccomended susbet r of elements e, where r is modified by computing the similarity of e to target t, an element in e.

    state
    elements: set Element
    vector: Element -> list Integer
    reccomended: list Element
    target: one Element
    proximity: list Integer

    actions
    pushup(t: Element, recommend: list Element)
        let similarElements = { e ∈ elements | e.vector ∈ proximity of t.vector }
        recommended := similarElements ∪ recommended

    pushdown(t: Element, recommend: list Element)
        let similarElements = { e ∈ elements | e.vector ∈ proximity of t.vector }
        recommended := similarElements - recommended
        recommended := similarElements - t

    reset() 
    ~~~


### Synchronizations

To inform what synergies, synchronizations, and objects APPNAME needs, I consider a sequence of events, from a user’s perspective (implies synergy with sessioning), that explores the full functionality of the app:

~~~
app APPNAME
    include Authenticating[Account]
    include Sessioning[Account]
    include Labeling[Activity, Account, Time Blocks]
    include Recommending[Activity, Account]
    include Requesting[Invitation, Follow]
    include Syncing[Time Block]
    
    include 
~~~

1. Logs in 
    + **SYNERGY:** authenticate and start a session 

2. Approves a follow request from an online friend 
    + **SYNERGY:** approve, add friend to unlabeled circle 
    ~~~
    sync acceptFollow(a2: Account):
        let a1 = Sessioning.getUser(a1)
        Requesting.accept(a2.FollowRequests, a1)
        Labeling.addLabel(a2, a1.defaultCircle)
    ~~~

3. Adds their friend to a specific circle 
    ~~~
    sync circle(a2: Account, circleName: String):
        let a1 = Sessioning.getUser(a1)
        Labeling.addLabel(a2, a1.circleName)
    ~~~

4. Reserve spots in recommended share for different circles
    ~~~
    sync reserveReccomendations(spots: Int, circleName: String):
        let a1 = Sessioning.getUser(a1)
        Labeling.addLabel(a1.shareRecommendations[0: spots], a1.circleName)
    ~~~

5. Sync GCal with APPNAME
    ~~~
    sync syncCalendar(c: Calendar):
        let a1 = Sessioning.getUser(a1)
        Syncing.addSource(a1.Calendars, Calendar)
        Syncing.pull(a1.Calendars)
    ~~~

6. Designates blocks in APPNAME’s calendar as free, hard commitment, or soft commitment 
    ~~~
    sync designateBlocks(t: Time Block, l: {"Free", "Hard Commitment", "Soft Commitment"}):
        let a1 = Sessioning.getUser(a1)
        Labeling.addLabel(t, a1.l)
    ~~~

7. Browses their activity feed, finds something they’ve been meaning to try, and pins it to Wishlist 
    + **SYNERGY:** label activity as part of the Wishlist collection, pushup similar activities in the activity feed 
    ~~~
    sync wishlist(a: Activity):
        let a1 = Sessioning.getUser(a1)
        Labeling.addLabel(a, a1.Wishlist)
        Reccomending.pushup(a, a1.Reccomendations)
    ~~~

8. Browses their activity feed, finds something they really don’t like, and indicates that they are Not Interested 
    ~~~
    sync notInterestsed(a: Activity):
        let a1 = Sessioning.getUser(a1)
        Reccomending.pushdown(a, a1.Reccomendations)
    ~~~

9. Accepts an invitation from another user 
    + **SYNERGY:** accept invitation, label time block in APPNAME’s calendar as hard commitment, sync up with iCal and gCal 
    ~~~
    sync acceptInvitation(i: Invitation):
        let a1 = Sessioning.getUser(a1)
        Requesting.accept(i, a1)
        Labeling.addLabel(i.timeBlock, a1."Hard Commitment")
    ~~~

10. Sees recommended shares 
    + **SYNERGY:** system recommends the best matched users by circle (query label)  

11. Sends an invitation to a queue of other users 
    + **SYNERGY:** error if a user in queue has a hard commitment (query label), otherwise send invitation 
    ~~~
    sync sendInvitation(i: Invitation):
        let a1 = Sessioning.getUser(a1)
        for a2 ∈  i.queue:
            if getLabel(i.timeBlock, a2."HardCommitment") then
                throw Error
        Requesting.send(i)
        Expiring.allocate(i, i.time)
    ~~~

12. Invitation expires (No synergy with sessioning) 
    ~~~
    sync expireInvitation(i: Invitation):
        Expiring.expire(i)
    ~~~

13. Session expires
    ~~~
    sync expireSession(a1: Account):
        Expiring.expire(a1)
        Sessioning.end(a1)
    ~~~

13. Logs out 
    ~~~
    sync logout(a1: Account):
        Sessioning.end(a1)
    ~~~

### Dependency Diagram

![image](/../assets/images/A3/DependencyDiagram.jpg)

## Wireframe

Prototyped in [Figma](https://www.figma.com/proto/EQOdPCVs9xuZRZL0DGacCc/Untitled?node-id=1-6&t=h29Yifh6pwXSeSDn-1).

## Design Tradeoffs

**To chat or not to chat?** 
+ A chat feature was part of my divergent design ideation. Users should be able to chat about additional logistics ad hoc, like if someone is running late, or if they want to share photos after spending time together. 
+ There is a Pandora’s Box of privacy, safety, and moderation that opens when features like chatting and commenting become available. Given time and resource constraints, I would not be able to flesh out the ethical details of such a feature to my satisfaction. More importantly, I considered the modularity of the app itself – can APPNAME stand alone without a chat feature? Users should connect online on another platform. When they send an invitation, there should be an option to send a message on the other platform to initiate conversations about any ad hoc logistics. 


**No creator circles**
+ While feature brainstorming, I thought very hard about how to serve the subset of my user base that feels a deep parasocial connection with a content creator and does not have similar online connections with other average users. I landed on a creator circle, where anyone who engages with a specific creator’s content can add themselves. They can request to follow others in the circle who are located within a fixed mile radius of them. 
+ This “solution” not only suggests location as an arbitrary constraint for an offline connection, when an offline connection can also be a phone call, Netflix party, or different activity beyond interactions on a familiar social network. It similarly surfaces concerns about safety, privacy, and moderation. Users who aren’t actually interested in a specific creator could still join their creator circle, because there is no way of ascertaining their interest level, which defeats the purpose of this feature. 

 

**No more synergy**
+ There is a familiar synergy between recommending and requesting, where users discover and send follow requests to each other because they are recommended to do so. Most platforms will base initial recommendations off the phone numbers in a user’s contacts that are associated with account credentials. 
+ Instead, I elected not to recommend users to each other. This app’s purpose is to transform how the average user, especially consumer, thinks about social media – as a means of connection. I can modularize the app to fit it onto the end of any social media app in a pipeline from online to offline connection. Users should find online connections on APPNAME from a link to their APPNAME account in the bio of another platform, like VSCO on Instagram. 