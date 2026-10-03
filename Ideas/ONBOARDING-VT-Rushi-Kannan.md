# Tenali Contributor Onboarding Document

**Contributor:** V T Rushi Kannan  
**GitHub:** [Rushikannan2](https://github.com/Rushikannan2)  
**Repository:** [github.com/vicharanashala/tenali](https://github.com/vicharanashala/tenali)  
**Date:** 3 October 2026

---

## 1. What is Tenali?

Tenali is a web-based learning platform focused mainly on mathematics. From using the project, I found that it is more than a normal question-and-answer website. The learning activities are interactive, and the platform is designed so that students can practise different kinds of problems and move through levels as they improve.

A student can work on mathematical puzzles, answer questions, get immediate feedback and, in several modules, see explanations or continue to the next level. The repository also contains features such as the Battle Arena and a code playground, so there are different ways of learning and practising inside the same project.

What I found interesting is the effort to make practice feel more like an activity than a static worksheet. The project has many different learning modules, and several of them use their own interaction and progression logic.

## 2. What do you understand by Tenali as a system?

I understand Tenali as a web application with a React frontend, a Node backend and MongoDB for persistent data.

The frontend is built with React and Vite. A lot of the main navigation logic is handled in `client/src/App.jsx`, where the application decides which module or screen to show. Other learning experiences are implemented as separate components and exercise files.

A typical learning flow is:

student opens a module → a question is selected or generated → the student interacts with it → the answer is checked → feedback is shown → progress is updated → the student moves to the next question or level.

For the regular question-based modules, the repository uses endpoints such as `/topic-api/question` to get a question and `/topic-api/check` to check an answer. Progress is also handled through API calls, while some individual modules keep their own local state and browser storage for things such as level progression.

The backend also contains authentication and persistent user data. I found bcrypt-based password handling, JWT authentication and MongoDB/Mongoose usage.

Different modules have different state and interaction patterns. For example, the Battle Arena uses Socket.IO for multiplayer communication. Schema Classifier has level unlocking and browser navigation. Equation-to-Story has a timer, drag-and-drop interaction and answer feedback. EquationCraftingLab works with equation blocks that the student selects and combines.

While tracing these modules, I noticed that seemingly small state-management choices can affect what the student actually sees. That was the main connection between my repository exploration and the issues I chose to investigate.

## 3. Current State of the Repository — What Has Been Done So Far

The repository is divided mainly into a `client/` frontend and a `server/` backend.

The frontend uses React and Vite and contains a large number of learning modules. The areas I explored most closely were the Vachana exercises, EquationCraftingLab, Schema Classifier and Equation-to-Story. There are also shared UI pieces, authentication screens, the Battle Arena and the code playground.

The backend contains the server setup, question and answer-checking APIs, authentication, progress handling and other application routes. MongoDB is used through Mongoose for persistent application data.

The project also has different types of user-facing flows. Some are regular question-based quizzes, while others have custom interfaces and their own state transitions. The Vachana modules, for example, have an overview page followed by level-specific exercises.

For development and validation, the client has ESLint and a Vite build setup. The repository also contains Vitest-related configuration and server-side tests. At the moment, the client does not have a normal `test` script in `client/package.json`, so I did not add a new testing framework just for the issues I was investigating.

I also checked the contribution workflow before making changes. Contributions are expected to start from existing GitHub issues, and a contributor must submit the onboarding document in the `Ideas/` folder before the first contribution PR.

## 4. Gaps Observed in the Code

The three main gaps I investigated are existing GitHub issues #263, #262 and #261. I checked the source files directly rather than relying only on the older error counts written in the issue titles.

### Issue #263 — `client/src/EquationCraftingLab.jsx`

The main problem I found was in the way the crucible selection state was updated after the blocks changed.

When the crucible reached the exactly-two-block state, the original code could briefly render two blocks with no selected block before the selection state caught up. In the browser trace this appeared as a short `2 blocks / 0 selected` state followed by `2 / 2`.

The final state was correct, but the intermediate render can make the interface flicker or look inconsistent to the student.

The file also contained redundant regular-expression escapes in `SAFE_MATH_REGEX` and a React purity lint issue around the way the affected functions were arranged. These were smaller cleanup items in the same issue.

### Issue #262 — `client/src/vachana/exercises/EquationToStory.jsx`

The main issue here was the lifecycle of the quiz timer.

The interval handle was being stored in state and then mutated. When a student left the level and entered it again, the old interval could remain active and another interval could be created. That meant more than one timer could run at the same time.

In the original behavior, the timer became roughly twice as fast after a restart, and the browser checks also found timer intervals still active after leaving the level.

This matters directly to the exercise because the student can lose time much faster than intended, and background timer work can continue after the student has already left the level.

The same file also had a purity-related use of `Date.now() + Math.random()` for temporary animation IDs.

### Issue #261 — `client/src/vachana/exercises/SchemaClassifier.jsx`

The issue here was mainly related to derived state and navigation.

The original implementation updated the schema options through an effect, and the level/navigation logic could allow the arena for a locked level to render briefly before redirecting the student to an allowed level.

The first-paint browser trace reproduced that behavior when a locked level URL was opened directly.

I also found that the option order could change during the first render/update cycle. The options themselves were correct, but the order could briefly change while the page was settling.

Both behaviors are small, but they are visible to the user. A locked level should not appear even briefly, and an option list should stay stable once it is shown.

## 5. Ideas for the Project

One improvement I would like to continue exploring is reducing state that is only being used to mirror other state. When a value can be derived directly from something such as the active level, calculating it directly or with `useMemo` is easier to follow than keeping another state value and synchronizing it with an effect.

For modules with timers, I think interval handles should have a clear lifecycle separate from normal UI state. The timer should start when the level starts and be cleared when the level ends or the student leaves. That makes restart behavior easier to reason about and avoids duplicate intervals.

For navigation-heavy modules, I would also keep the decision about whether a level is accessible separate from the code that synchronizes the browser URL. This would make it easier to ensure that a locked level is never rendered before the navigation correction happens.

More generally, I think the frontend would benefit from making state ownership more explicit in the larger exercise components. The issues I found showed that a small effect or mutable value can have consequences that are not obvious from the final state alone.

## 6. Your Contribution

During my onboarding, I investigated three existing frontend issues: #263, #262 and #261. I read the affected components, removed the related ESLint suppressions in isolated copies, traced the relevant state and event flows, and reproduced the reported behavior in browser-based tests.

I prepared separate fixes for the three issues so that they could be tested independently.

For #263, I moved the crucible selection update into the event flow that changes the crucible, removed the redundant regular-expression escapes, and adjusted the affected function ordering.

For #262, I changed the timer handle and the temporary float ID counter to refs so they are treated as mutable resources rather than state values.

For #261, I changed `optionsByQuestion` to be derived from the active level, moved the locked-level clamp into initialization, and changed the navigation synchronization so that it updates browser history without introducing another state-setting effect.

I then ran browser regression checks against both the fixed and original versions in separate sandboxes. The fixed versions passed the automated checks I prepared for these issues, and the original versions reproduced the corresponding problems. I also checked the affected files with ESLint and built the client successfully in the isolated sandboxes.

At this stage, these fixes are prepared and validated, but they have not been merged into the Tenali repository yet. I plan to submit the fixes as three separate issue-linked pull requests, one for each issue, after the onboarding document is reviewed.
