# Reacteractions - Reactive Discord Interactions

A system for creating interactive Discord messages based on interactions using a Reactive model.

The inspiration of using React directly came from a similar package [reacord](https://github.com/itsMapleLeaf/reacord), however this package aims to be more of a full-fledged replacement for command handling systems.

## Intended Features / Progress

- [ ] Slash commands modelled by functional React components (with JSX)
  - [ ] Component initialisation, rendering, and expiring
  - [ ] Slash command options passed through props (and maybe autocomplete?)
  - [ ] Connector utilities to register slash commands
  - [ ] Possibly a nextjs-like command organisation system based on folder & file structure
- [ ] React hook support, mainly `useState`, `useEffect` and `useContext`
- [ ] Hydration
  - [ ] Connector utilities to dehydrate/hydrate
  - [ ] External state management in redis, extensible with anything
  - [ ] Static Message Generation, if it proves to be worthwhile for performance
- [ ] Custom hooks
  - [ ] `useController` for general control utilities
  - [ ] `useHydrator` to enable hydration and hydration utilities
  - [ ] `useModal` for showing & collecting modal inputs
  - [ ] `useMessage` for showing additional messages such as an ephemeral response

Connector utilities library support:
- [ ] discord.js
- [ ] eris
- [ ] discordeno
- [ ] detritus

## Usage Examples (subject to change)

The following usage examples are intended to guide development and may change.

```tsx
function MyCounter() {
  const [count, setCount] = useState(0)

  return <>
    <Embed>
      <Embed.Title>This button has been clicked {count} times</Embed.Title>
    </Embed>
    <Button onClick={() => setCount(count + 1)}>+1</Button>
  </>
}
```

```tsx
// Dehydration is when the bot goes offline and cannot respond to interactions.
// Rehydration is when the bot returns online and can respond again.

// Using hydration in a way like this would not be recommended for large bots, since you'd be editing
// heaps of messages and probably hit ratelimits.

function MyHydratedCancellableCounter() {
  const { deactivate, isDeactivating } = useController()
  const [count, setCount] = useState(0)

  // A re-render would occur with isDehydrating set to true when the bot is going offline
  const { isDehydrating } = useHydrator()

  return <>
    <Embed>
      <Embed.Title>This button has been clicked {count} times</Embed.Title>
      {botIsOffline && <Embed.Footer>Bot offline :(</Embed.Footer>}
      {isDeactivating && <Embed.Footer>Message deactivated :(</Embed.Footer>}
    </Embed>
    <Button
      onClick={() => setCount(count + 1)}
      disabled={botIsOffline}
    >
      +1
    </Button>
    <Button
      style="danger"
      onClick={() => deactivate({ removeComponents: true })}
    >
      deactivate
    </Button>
  </>
}
```

Unless I think of a neat way to do it otherwise, slash command options and autocomplete would only be supported using the folder-file command structure system. That would look something like the following.

```tsx
// src/commands/counter.js

// A command to count up to a given number, or infinitely. Autocomplete suggests possible powers of 2 which the user may want to count to.

export const name = 'counter'
export const description = 'count up to a number!'
export const options = [{
  type: 'INTEGER',
  name: 'number',
  description: 'number to count to',
  required: false,
  autocomplete: true
}]

export const autocomplete = {
  // Autocomplete functions are mapped by option name. Could also maybe just be put in the option 'autocomplete' value itself.
  number: userValue => {
    const powersOfTwo = Array.from({ length: 20 }, (_, i) => 2 ** i)
    const userNumber = Number(userValue)
    if (userNumber) {
      return powersOfTwo.filter(powerOfTwo => powerOfTwo > userNumber)
    }
  }
}

export default function Counter({ options: { number: target } }) {
  const { deactivate, isDeactivating } = useController()
  const [count, setCount] = useState(0)

  if (count === target) {
    deactivate({ removeComponents: true })
  }

  return <>
    <Embed>
      <Embed.Title>This button has been clicked {count} times</Embed.Title>
      {/* Change the foter depending on if it's been deactivated or the target has been reached */}
      {isDeactivating
        ? (count >= target
          ? <Embed.Footer>You reached the goal!</Embed.Footer>
          : <Embed.Footer>You stopped counting {target && 'to ' + target}</Embed.Footer>)
        : <Embed.Footer>You're counting {target ? 'to ' + target : 'infinitely'}!</Embed.Footer>}
    </Embed>
    <Button onClick={() => setCount(count + 1)}>+1</Button>
    <Button
      style="danger"
      onClick={() => deactivate({ removeComponents: true })}
    >
      deactivate
    </Button>
  </>
}

