---
title: A5 alpha
layout: doc
---

# A5: Frontend Design (Alpha)

## (*Prominent*)Links

+ **Frontend Code Repo:** [https://github.com/parikhushii/frontend](https://github.com/parikhushii/frontend)
    - full, working implementation of **Circling**
+ **Deployed Service** [https://quest-phi-jet.vercel.app/](https://quest-phi-jet.vercel.app/)

## UI Heuristic Recommendations for Quest App

### 1. Discoverability
- **Current issue**: Users navigate multiple steps to find a user in the Venn diagram structure, which may be confusing and time-consuming.
- **Recommendation**: Implement a search option, enabling users to directly look up a person. This reduces the learning curve, improving how quickly users understand and navigate the interface.

### 2. Efficiency
- **Current issue**: The current multi-step process for finding a user slows down task completion.
- **Recommendation**: Streamline navigation with shortcuts and accelerators, such as keyboard shortcuts for expert users or a quick access sidebar that reduces clicks to reach key functionality.

### 3. Safety & Error Tolerance
- **Current issue**: Users can't undo requests once submitted, leading to potential mistakes.
- **Recommendation**: Add a forgiveness period that allows users to cancel or revise their requests. This could be visualized with a countdown timer and synchronized with an expiration concept, ensuring error tolerance while maintaining control over user actions.

### 4. Security
- **Current issue**: The app should safeguard user privacy and integrity.
- **Recommendation**: Ensure all sensitive actions like sending requests have confirmation dialogues and clear, understandable privacy terms. Secure user data through encryption and protect interaction history to uphold confidentiality.

### 5. Pleasantness & Accessibility
- **Current issue**: Consider the overall aesthetics and inclusivity of the app.
- **Recommendation**: Use calming color schemes, intuitive iconography, and ensure that visual and auditory cues are accessible for users with disabilities. Incorporate elements like adjustable font sizes, screen reader compatibility, and touch gesture-friendly layouts.

### 6. Physical Heuristics
- **Fitt’s Law**: Make frequently used elements, such as the search bar, easy to access and within reach. Place larger targets for critical actions like sending requests.
- **Gestalt Principles**: Ensure the interface visually groups related elements together. For instance, users within a certain status in the Venn diagram should be color-coded or grouped to signify their similarity or relevance.
- **Situational Context**: Highlight the user’s current position in the Venn diagram, ensuring the app provides clear context on where they are and the app's current state.

### 7. Linguistic & Consistency Considerations
- **Current issue**: Ensure the app uses simple, user-friendly language that avoids technical jargon.
- **Recommendation**: Messages should be intuitive and informative, written in plain language that doesn’t require technical knowledge. Keep names, symbols, and icons consistent throughout the app, aligning them with industry standards to reduce cognitive load.
