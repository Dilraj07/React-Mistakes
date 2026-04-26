1. 🚩 Using libraries for what vanilla JS can do
We've all done it:

Importing lodash just to use isNil instead of == null.

Using dayjs rather than Intl.DateTimeFormat for future use cases (spoiler alert: they end up never coming).

Etc.

But this is wrong because every dependency:

adds maintenance overhead: You have to update it when bugs are fixed, vulnerabilities disclosed, new versions come out.

adds to your bundle size.

can break when you upgrade React or TypeScript.

Before you add a dependency, ask yourself: can I do this with vanilla JS?

If the answer is yes, do that instead. It keeps your codebase lean.

2. 🚩 Reaching for heavy dependencies when lighter alternatives exist
moment.js is 75KB minified. day.js is 3KB with the same API.

axios is 14KB. The native fetch API is free.

I've reviewed PRs where someone added a 70KB library to do something a 3KB alternative handled just as well.

Nobody checked the size, because nobody had the habit. It adds up fast and your users on a 3G connection in Lagos or São Paulo are the ones who pay for it.

Before you install, check the bundle size on bundlephobia.

If a lighter alternative exists, use it.

3. 🚩 No linter or formatter configured
Non-negotiable.

I once sat through a meeting where engineers spent 30 minutes arguing about whether to use tabs or spaces.

What settled the debate? Configuring the formatter for the entire repo and moving on.

Without repo-level config, every review becomes a style debate instead of a real conversation about the code.

Set it up once and move on.

4. 🚩 No consistent folder structure
I've wasted so many hours of my life trying to figure out where to put a new file 🥲.

One developer organises by feature, another by file type, a third just dumps things wherever.

Pick a convention – feature-based, domain-based, whatever works for your team – document it, and enforce it in reviews.

// 🚩 Red flag: chaotic structure
src/
  components/Button.tsx
  pages/home/helpers.ts
  utils/userStuff.ts
  lib/api.ts
  shared/Button.tsx   // wait, there's already a Button?
5. 🚩 Junk drawer utils files
Every codebase has one. A utils.ts that starts with 3 helper functions and grows into 40 unrelated ones.

Nobody owns this file.

Everyone adds to it.

Nobody cleans it.

A simple change to one function can cause merge conflicts with a change to a completely unrelated function.

A small change can cause bugs in an unrelated function because they share the same scope.

In the example below, formatDate and calculateTax are neighbors for no reason.

// utils.ts — the junk drawer
export function formatDate(d: Date) { /* ... */ }
export function parseQueryString(s: string) { /* ... */ }
export function calculateTax(amount: number) { /* ... */ }
export function debounce(fn: Function, ms: number) { /* ... */ }
export function hexToRgb(hex: string) { /* ... */ }
// ...35 more
Instead, related functions should be grouped together: date-utils.ts, string-utils.ts, math-utils.ts.

If your utils file needs a table of contents, it's time to break it up.

6. 🚩 Related files not colocated
When a component lives in /components, its styles in /styles, its tests in /__tests__, and its helpers in /utils... you're constantly jumping across the project to understand one feature.

Files that change together should live together.

When I need to modify the UserProfile feature, I want to open one folder, not four. And when the feature gets killed, I want to delete one folder, not hunt through four directories for orphaned files.

PS: I’m not jumping that much between files these days: Claude does 😅. But still…

// 🚩 Red flag: scattered files
src/
  components/UserProfile.tsx
  styles/UserProfile.css
  __tests__/UserProfile.test.tsx
  utils/userProfileHelpers.ts

// ✅ Colocated
src/
  features/user-profile/
    UserProfile.tsx
    UserProfile.css
    UserProfile.test.tsx
    helpers.ts
7. 🚩 Barrel files that re-export everything
I've done that mistake too 🙈.

You want to simplify imports, so you create an components.ts file that re-exports everything in the folder.

Little did you know that this creates a hidden coupling between all those files.

In development, your bundler processes the entire barrel on every hot reload, even files you never touch.

You get weird compilation errors due to a file that you're not even using but is still being processed.

You have a hard time tracing where something is defined: you click through to the import and land in a wall of re-exports instead of the actual code.

If you must use barrel files, be explicit: name each export instead of using export *.

// 🚩 components.ts
export * from './Button'
export * from './Modal'
export * from './Table'
// ...50 more
8. 🚩 God components
Data fetching, state management, form validation, error handling - all in one file, all in 500+ lines.

Signs you have a God component:

Scrolling 30 seconds to find the return statement

Searching return gives you 20+ results

I've worked on components like this: you fix a bug in the form validation and accidentally break the data fetching because everything shares the same scope.

So break it up. Each piece should have one job.

9. 🚩 Passing entire objects when only one field is needed
// 🚩 Red flag
<UserAvatar user={user} />

// ✅ Better
<UserAvatar avatarUrl={user.avatarUrl} name={user.name} />
When you pass the whole object, the child component is coupled to the shape of user and re-renders whenever any field on that object changes, not just the ones it uses.

You also make the component harder to reuse: what if another part of the app has avatar data but it doesn't live in a user object?

So only pass components the data they need. It keeps them decoupled, reusable, more performant, and easier to test.

10. 🚩 Storing derived values in state
// 🚩 Red flag
const [firstName, setFirstName] = useState('')
const [lastName, setLastName] = useState('')
const [fullName, setFullName] = useState('')

useEffect(() => {
  setFullName(`${firstName} ${lastName}`)
}, [firstName, lastName])

// ✅ Just compute it
const fullName = `${firstName} ${lastName}`
If a value can be calculated from other state or props, it's not state, it's a computation.

I've seen this cause real bugs: there's always a render where fullName is stale because the effect hasn't run yet.

The fix is simple: just compute it inline.

If the computation is expensive, wrap it in useMemo. But don't store it in state.

11. 🚩 Using state when it should be a ref
Timer IDs, previous values, flags that shouldn't trigger a re-render: these belong in a useRef, not useState.

If changing the value shouldn't update the screen, it's not state.

// 🚩 Red flag: triggers a re-render every time
const [timerId, setTimerId] = useState(null)

// ✅ No re-render needed
const timerIdRef = useRef(null)
12. 🚩 Putting everything in a global store
Not everything needs to be in Redux or a global Context.

I've worked on codebases where tooltip visibility was stored in Redux: an action to open it, a reducer to track it, a selector to read it, all for state that only one component ever needed.

// 🚩 Red flag: global state for local UI
dispatch(setTooltipVisible(true))

// ✅ Keep it where it lives
const [isVisible, setIsVisible] = useState(false)
The rule: if no other part of the app needs to know about it, keep it local.

13. 🚩 A single massive Context that re-renders half the app
// 🚩 Red flag: one context for everything
const AppContext = createContext({
  user: null,
  theme: 'light',
  locale: 'en',
  sidebarOpen: false,
  notificationCount: 0,
})
Every time any of these changes, every consumer re-renders.

The sidebar toggles? Your notification badge re-renders. 

The locale changes? Your sidebar re-renders.

Split your contexts by concern: UserContext, ThemeContext, etc.

It's a small upfront cost that saves you from chasing phantom performance problems later.

And if your contexts start becoming unmanageable, that's a sign you need a state management library with more granular updates.

PS: If your app is simple enough, using a single context is fine 😉.

14. 🚩 Poor TypeScript hygiene
any everywhere. You're using TypeScript but typing things as any?
You're getting all the overhead with none of the safety.

That's just the start. Poor TypeScript hygiene also looks like:

No discriminated unions — you're allowing impossible states.

Switch statements without exhaustive checks — every new variant
is a bug waiting to happen.

TypeScript is only as good as the types you write.

If you’re new to TypeScript, check my post: TypeScript to know for React 😉.

// 🚩 Red flag: allows impossible states
type FormState = {
  isSubmitting: boolean
  isSuccess: boolean
  errorMessage: string | null
}
// { isSubmitting: true, isSuccess: true } — shouldn't be possible, but is

// ✅ Discriminated union
type FormState =
  | { status: 'idle' }
  | { status: 'submitting' }
  | { status: 'success' }
  | { status: 'error'; message: string }
15. 🚩 eslint-disable to silence the exhaustive-deps warning
// 🚩 Red flag
useEffect(() => {
fetchData(userId)
// eslint-disable-next-line react-hooks/exhaustive-deps
}, [])
This looks harmless. It's not.

The linter is telling you: your effect depends on userId, but it won't re-run when userId changes. 

Two things can go wrong:

Your users see stale data whenever userId changes.

Someone adds a dependency inside fetchData later, gets no warning, and the bug ships silently.

The fix isn't to silence the warning — it's to understand it:

Maybe fetchData should be wrapped in useCallback.

Maybe the effect should list userId as a dependency.

Silencing the linter doesn't fix the underlying issue — it just hides it until a user reports stale data.

16. 🚩 Using array indices as keys
We've all written this at some point:

// 🚩 Red flag
{items.map((item, index) => (
  <TodoItem key={index} item={item} />
))}
React just needs something for key, right?

Here's what goes wrong. 

You have a list of 3 items. The user deletes item 0. React sees key={0} still exists so it reuses the first DOM node instead of removing it. The component doesn't reset. The stale state from the old item bleeds into the new one.

You'll see this as: inputs that don't clear, animations that fire on the wrong item, form values that belong to a deleted row.

// ✅ Use a stable, unique id
{items.map((item) => (
  <TodoItem key={item.id} item={item} />
))}
If your data doesn't have an id, generate one when it enters your system, not at render time.

17. 🚩 Using a hook when a plain function would do
Just because you're building components doesn't mean everything has to be a hook.

If your function doesn't call React state, effects, or other hooks, it's just a function.

Making it a hook means it can only be called at the top level of a component or another hook, not inside event handlers, loops, or conditionals.

A plain function can be called anywhere. It's also easier to test 😉.

// 🚩 Red flag: it's a hook for no reason
function useFormatCurrency(amount: number) {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount)
}

// ✅ Just a function
function formatCurrency(amount: number) {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount)
}
18. 🚩 Rolling your own data fetching layer
// 🚩 Red flag
useEffect(() => {
  setLoading(true)
  fetch(`/api/users/${id}`)
    .then(res => res.json())
    .then(data => {
      setData(data)
      setLoading(false)
    })
    .catch(err => {
      setError(err)
      setLoading(false)
    })
}, [id])
This code works. For a simple one-off fetch, it's fine.

The red flag is what comes next.

Your app needs a loading state. 

You add one. 

Then error handling.

Then retry on failure. 

Then cache invalidation. 

Then request deduplication so the same request doesn't fire twice.

There's also a subtle bug already: if id changes quickly, the first response can overwrite the second, a race condition you'll need to handle with an AbortController.

You've just started rebuilding React Query. Poorly.

Libraries like React Query and SWR already solved all of this.

You don't have to use them — but if you find yourself adding more and more logic around useEffect + fetch, reach for a library instead of reinventing it.

19. 🚩 No error boundaries
I've seen an app where JSON.parse on a value with an unexpected shape crashed the entire app: navigation, sidebar, everything. Gone. 

TypeScript couldn't catch it: JSON.parse returns any, so the type system had no idea.

A single unhandled error shouldn't take down the whole page.

Wrap critical sections so the rest of the app keeps working. The user loses one capability instead of everything.

// ✅ Wrap critical sections
<ErrorBoundary fallback={<p>Something went wrong.</p>}>
  <UserDashboard />
</ErrorBoundary>
20. 🚩 Empty catch blocks
I can't tell you how many times I've seen this:

User complaints about an issue

You check the logs and see... nothing. No errors, no warnings, no clues at all.

You have to ship new code just to add logging and figure out what went wrong  🤦‍♀️.

The culprit is always the same:

// 🚩 Red flag
try {
  await saveData()
} catch (e) {}
At minimum, log the error. Better yet, tell the user something went wrong (+ how to fix it).

21. 🚩 No loading or error states
No loading spinner, no skeleton, no error message.

Just a blank screen when the API is slow and a crash when it fails.

Your users don't have localhost speeds and perfect networks. You need three states: loading, error, success.

// 🚩 Red flag: only handles success
function UserList() {
  const [users, setUsers] = useState([])

  useEffect(() => {
    fetch('/api/users').then(r => r.json()).then(setUsers)
  }, [])

  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>
}
22. 🚩 Accidentally breaking memoization with default values
I once introduced a new component to my codebase.

And for whatever reason, my app started crashing whenever I typed in an unrelated input.

The culprit? A ?? [] default value in the new component that was creating a new array on every render.

The fix:

// ✅ Define the default outside the component
const EMPTY_ITEMS = []
<ItemsList items={items ?? EMPTY_ITEMS} />
Same goes for {}, () => {}, and any other inline default. 

If it's passed as a prop and the child is memoized, define it outside the render.

23. 🚩 Unreadable conditional rendering
Nested ternaries and long && chains both make JSX impossible to follow:

// 🚩 Nested ternaries
{isLoading ? <Spinner /> : error ? <Error /> : data ? <Content /> : null}

// 🚩 Long && chains
{isLoggedIn && hasPermission && !isExpired && featureFlag && <SecretPanel />}
If you have to mentally parse boolean logic to understand what renders, extract it into a variable or use early returns:

// ✅ Clear and readable
if (isLoading) return <Spinner />
if (error) return <Error />
if (!data) return null
return <Content />
24. 🚩 No early returns
If you're five levels of indentation deep, flip your conditions and return early.

This makes the code easier to read.

// 🚩 Red flag: deeply nested
function UserProfile({ user }) {
  if (user) {
    if (user.isActive) {
      if (user.hasProfile) {
        return <Profile data={user.profile} />
      } else {
        return <CreateProfile />
      }
    } else {
      return <InactiveMessage />
    }
  } else {
    return <NotFound />
  }
}

// ✅ Early returns
function UserProfile({ user }) {
  if (!user) return <NotFound />
  if (!user.isActive) return <InactiveMessage />
  if (!user.hasProfile) return <CreateProfile />
  return <Profile data={user.profile} />
}
25. 🚩 Magic values with no explanation
I can't tell you how many times I've wanted to modify a value in the code but was too scared to change it because I had no idea why it was that value.

setTimeout(retry, 3000). Why 3 seconds?

pageSize: 37. Why 37?

If a value isn't self-explanatory, give it a name and document it.

// 🚩 Red flag
setTimeout(retry, 3000)
const PAGE_SIZE = 36

// ✅ Named and documented
const RETRY_DELAY_MS = 3000 // Matches the p95 API response time
const PAGE_SIZE = 36 // 4 columns × 9 rows in the grid layout
26. 🚩 The same conditional logic scattered everywhere
// 🚩 Red flag: scattered across the codebase
{user.role === 'admin' && <DeleteButton />}
{user.role === 'admin' && <AdminPanel />}
{user.role === 'admin' || user.role === 'moderator' && <ModTools />}

// ✅ Centralized
function canDelete(user: User) { return user.role === 'admin' }
function canModerate(user: User) { return ['admin', 'moderator'].includes(user.role) }
When if (user.role === 'admin') appears in 10 different components, you've built a maintenance trap.

The day someone adds a new role or changes what "admin" means, you don't want to hunt through 10 files.

Centralise it into permission helpers and import those everywhere.

27. 🚩 No abstraction layer over third-party libraries
// 🚩 Red flag: tightly coupled to recharts
import { LineChart, Line, XAxis, YAxis } from 'recharts'
// ...used directly in 20 components

// ✅ Wrapped behind your own interface
import { Chart } from '@/components/Chart'
<Chart type="line" data={data} xKey="date" yKey="revenue" />
If your charting library is imported directly in 20 components, you're one migration away from rewriting half the app.

A thin wrapper isn't always worth the effort for every dependency.

But for anything you use in more than a handful of places: charting, analytics, HTTP clients, etc. it pays for itself the first time you need to swap.

28. 🚩 Massive files nobody dares refactor
Every codebase has one.

The 1,200-line component that everyone is afraid to touch because there are no tests and everything might break. So it keeps growing. New features get bolted on. The fear compounds. Nobody refactors it. It just gets worse.

The fix is never as hard as the team fears.

Start small: extract one pure function. Write one test. Then another. You don't need to rewrite the whole thing in a weekend. You need to stop feeding it.

29. 🚩 Not memoizing context values
// 🚩 Red flag: new object on every render
<AuthContext.Provider value=>
  {children}
</AuthContext.Provider>

// ✅ Memoize the value
const value = useMemo(() => ({ user, login, logout }), [user, login, logout])
<AuthContext.Provider value={value}>
  {children}
</AuthContext.Provider>
Every time the provider re-renders, it creates a new value object.

Every consumer re-renders, even if nothing changed. If your context is near the top of the tree, that's potentially your entire app re-rendering for nothing. One line of useMemo prevents it.

None of these flags mean a codebase is doomed. 

Every one of them is fixable. 

I've introduced some of them myself: the first time you see a pattern is rarely the first time you've written it.

But if you're counting more than a handful in the same project, that's worth paying attention to. Not as a judgment, but as a signal that the codebase needs some care.

Start with the ones that hurt the most. Fix those. The rest will get easier.

That's a wrap 🎉

Leave a comment 📩 to share the red flags you've spotted in the wild.

And don't forget to drop a '💖🦄🔥'.

If you're learning React, download my 101 React Tips & Tricks book for FREE.

If you like articles like this, join my FREE newsletter, FrontendJoy.


🤖 AI TIP
Use the skill-creator skill to create new skills and improve existing ones.


Reply

Newest first
Avatar
Add your comment...

Login or Subscribe to participate

Keep Reading
17 Tips from a Senior React Developer
Jan 6, 2025

•

12 min read

17 Tips from a Senior React Developer

Ndeye Fatou Diop
Ndeye Fatou Diop
✨ Is React really that great?
Jun 22, 2025

•

4 min read

✨ Is React really that great?

Ndeye Fatou Diop
Ndeye Fatou Diop
Is React as hard/complex as it sounds?
Dec 3, 2024

•

9 min read

Is React as hard/complex as it sounds?
7 Reasons Why React Feels Hard (and How to Fix It)

Ndeye Fatou Diop
Ndeye Fatou Diop

View more
FrontendJoy
FrontendJoy
Learn how to become a Senior Frontend Engineer without a CS Degree or natural talent.

Enter Your Email
Join Free
Home
Posts

About
About

Freebies
101 React Tips & Tricks E-Book

© 2026 FrontendJoy.
Report abuse
Privacy policy
Terms of use
Powered by beehiiv


