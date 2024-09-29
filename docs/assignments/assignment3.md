---
title: A3
layout: doc
---

# A3: Convergent Design

## Pitch

## Functional Design

### Collapsing Features into Concepts

### Concepts

1. **Authenticating**
    + **Purpose**
    + **Operational Principle**
    + **State**
    + **Actions**
2. **Sessioning**
    + **Purpose**
    + **Operational Principle**
    + **State**
    + **Actions**
3. **Labeling**
    ~~~
    Labeling[Element]

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

    queryLabel(l: Label) -> set Element
        return l.elements
    
    ~~~
4. **Requesting**
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
5. **Syncing**
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

    pull()
        let sources = { ds ∈ datasets | ds.isSource = true }
        let targets = { ds ∈ datasets | ds.isSource = false }
        
        for each source s ∈ sources do
            for each d ∈ s.data do
                if d ∉ t.data for each target t ∈ targets then
                    t.data := t.data ∪ {d}

    push()
        let sources = { ds ∈ datasets | ds.isSource = true }
        let targets = { ds ∈ datasets | ds.isSource = false }
        
        for each target t ∈ targets do
            for each d ∈ t.data do
                if d ∉ s.data for each source s ∈ sources then
                    s.data := s.data ∪ {d}

    addSource(s: Dataset)
        s.isSource := True
        datasets := datasets ∪ {s}

    addTarget(t: Dataset)
        t.isSource := False
        datasets := datasets ∪ {t}
    ~~~
6. **Recommending**
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
    pushup(t)
        let similarElements = { e ∈ elements | e.vector ∈ proximity of t.vector }
        recommended := similarElements ∪ recommended

    pushdown(t)
        let similarElements = { e ∈ elements | e.vector ∈ proximity of t.vector }
        recommended := similarElements - recommended
        recommended := similarElements - t
    ~~~


### Synchronizations

A user saves actvity A to tgier favorties collection


### Dependency Diagram