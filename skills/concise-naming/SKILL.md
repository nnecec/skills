---
name: concise-naming
description:
  Enforces clear, concise naming for all code identifiers — hooks, components,
  functions, interfaces, and any code being written or renamed. Shortens verbose
  names (getUserDataFromId → getUser) and strips redundant prefixes/suffixes
  (Data, Object, Info, Event, etc.). Does NOT trigger when the name is already
  concise or the task is purely conceptual without code.
metadata:
  version: '1.0'
---

# Concise Naming

Name things with the minimum number of words. Brevity does not mean illegible —
keep names grammatical and unambiguous.

## Core Principle

If a name works with one fewer word, drop it.

## Redundant Words to Remove

These words rarely add meaning and should usually be stripped:

| Remove                | Reason                                                                                                                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Data`                | Types are already typed; variables already hold data                                                                                                                |
| `Config`              | Often redundant when the thing is obviously configuration; **keep** when an entity and its settings coexist (`user` vs `userConfig`)                                |
| `Manager` / `Handler` | Often just a class doing its job                                                                                                                                    |
| `Info`                | Same as Data                                                                                                                                                        |
| `From`                | Implied by context (`getUser(id)` not `getUserFromId`)                                                                                                              |
| `With` / `And`        | Split into separate functions or pick the dominant concern                                                                                                          |
| `Only`                | Redundant modifier                                                                                                                                                  |
| `that` / `this`       | Obvious from scope                                                                                                                                                  |
| `Object` / `Item`     | Vague and empty                                                                                                                                                     |
| `By` / `ById`         | Often unnecessary for simple parameters (`getUser(id)` not `getUserById`); **keep** when the name encodes a map, index, or group-by (`usersById`, `labelsByLocale`) |
| `Query`               | For hooks already prefixed `use`, `Query` is redundant                                                                                                              |
| `Inline`              | Obvious from context                                                                                                                                                |

## Concise by Default

- Use short, common words. If a long word can be replaced by a short one with
  the same meaning, do it.
- **Short word first, when semantics allow.** `get` over `retrieve`, `check`
  over `determine`, `do` over `perform`, `find` over `locate`, `show` over
  `display`.
- `use` is already short — keep it. But anything after `use` should be as lean
  as possible: `useUser` beats `useUserData`.

## Decision Rules

### Rule 1: Drop redundant suffixes

```
bad:  userData              good: user
bad:  configData             good: config
bad:  settingsInfo           good: settings
bad:  useUserDataQuery       good: useUser
bad:  useEmoticonConfigQuery good: useEmoticon
bad:  cacheManager           good: cache
bad:  clickEventHandler      good: handleClick
bad:  MentionOnlyPattern     good: MentionPattern
```

### Rule 2: Prefer single verbs for getters/setters

```
bad:  getUserById           good: getUser
bad:  fetchMessageList      good: fetchMessages
bad:  removeItemFromList    good: removeItem
bad:  performUserUpdate     good: updateUser
```

### Rule 3: Split names with "And"/"With" into separate concerns

```
bad:  renderTextWithMentionsAndEmoticons   good: renderText (or split into renderMentions, renderEmoticons)
bad:  useUserAndPostData                    good: useUser, usePost
bad:  validateAndSave                       good: validate, save
```

### Rule 4: Keep words that disambiguate

```
bad:  cache           good: diskCache   (if there is also memoryCache)
bad:  getUser         good: getUserFromDb   (if ambiguous otherwise)
```

Context determines what is truly redundant. If dropping a word creates genuine
ambiguity, keep it.

## Grammar Check

Concise does not mean ungrammatical. Watch for:

- Plurals for collections: `messages` not `messageList`
- Verb vs noun: `fetchUser()` not `getUserData()`
- Adjectives only when they distinguish variants: e.g. `remoteUsers` when local
  users also exist; avoid vague compounds like `filteredUserList` when a single
  precise modifier would do

## When to Shorten vs Keep

- **Shorten** when removing a word changes nothing
- **Keep** when the word carries necessary meaning or prevents ambiguity
- **Split** when a name has multiple concerns

### Keep these suffixes when they disambiguate

- **`Config`** — Same scope has both the thing and how it is configured (e.g.
  `User` + `UserConfig`, or `loadUser` vs `loadUserConfig`).
- **`ById` / `By…`** — The value is keyed or grouped: a map from id to row, a
  record of slices per key, etc. Dropping `ById` would make `users` look like a
  flat array when it is really `Record<UserId, User>`.

## Workflow

1. After naming something, ask: "Does it work without one of these words?"
2. If yes, remove it
3. Verify the short name is still unambiguous in context

## Examples

| Bad                                  | Good             | Why                                                          |
| ------------------------------------ | ---------------- | ------------------------------------------------------------ |
| `renderInlineEmoticon`               | `renderEmoticon` | `Inline` is redundant                                        |
| `renderTextWithMentionsAndEmoticons` | `renderText`     | Context makes it obvious                                     |
| `mentionOnlyPattern`                 | `mentionPattern` | `Only` is redundant                                          |
| `getMessageById`                     | `getMessage`     | `ById` is obvious                                            |
| `userObject`                         | `user`           | `Object` is vague and redundant                              |
| `handleClickEvent`                   | `handleClick`    | `Event` is redundant                                         |
| `fetchUserList`                      | `fetchUsers`     | `List` is redundant                                          |
| `onClickHandler`                     | `onClick`        | `Handler` is redundant                                       |
| `useUserDataQuery`                   | `useUser`        | `use` already signals hook; `Data` and `Query` are redundant |
| `useEmoticonConfigQuery`             | `useEmoticon`    | `Config` and `Query` are redundant with context              |
