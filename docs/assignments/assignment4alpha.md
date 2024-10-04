---
title: A4 alpha
layout: doc
---

# A4: Backend Design (Alpha)

## Some Important Links

+ **Backend Code Repo:** [https://github.com/parikhushii/fa-24-backend](https://github.com/parikhushii/fa-24-backend)
    - full, (mostly) working implementation of **Labeling** and **Requesting**
    - RESTful route outline
+ **Deployed Service** [https://fa-24-backend.vercel.app/](https://fa-24-backend.vercel.app/)

## Concepts

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

## Dependency Diagram

![image](/../assets/images/A3/DependencyDiagram.jpg)