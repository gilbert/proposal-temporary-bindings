# ESNext Proposal: Temporary Bindings

This proposal introduces a more readable way of assigning a variable to a complex value that involves multiple steps.

## Motivation

Take the following real-world example. Given an array of `group` objects, you want to calculate your user's access to each of those groups' parent organizations.

This will involve several steps:

```js
let orgIds = groups.map(g => g.org_ids);
let uniqueOrgIds = [...new Set(orgIds)];
let promises = uniqueOrgIds.map( async id => {
  let has_access = await Role.hasOrgPermission(id, user.id, perm);
  return { id, has_access };
})
let subresults = await Promise.all(promises);

let hasAccessByOrgId = objectify(subresults, { by: 'id' });
```

Our goal, `hasAccessByOrgId`, is drowned out by the large number of variables leading up to it. These other variables are temporary by nature – they only exist to break up the multi-step process, to pass data down the pipeline.

With the Temporary Bindings proposal, the above can be rewritten as follows:

```js
const($) hasAccessByOrgId =
  groups.map(g => g.org_ids),
  new Set($),
  [...$],
  $.map( async id => {
    let has_access = await Role.hasOrgPermission(id, user.id, perm)
    return { id, has_access }
  }),
  await Promise.all($),
  objectify($, { by: 'id' });
```

In the above code, `$` is set to be a temporary binding that only exists within the `const` assignment of `hasAccessByOrgId`. The name is determined by the programmer; we could have used `_`, `abc`, or any other valid identifier instead.

The temporary binding is first assigned to the first expression after the equals sign (`groups.map()`), then subsequently assigned to each comma-separated expression after that. The last expression (`objectify()`) is the end result – the final value to be assigned to `hasAccessByOrgId`.

## Benefits

- Pipeline with plain old JavaScript expressions.
- The temporary binding identifier is configurable ad-hoc, allowing the programmer to avoid shadowing if need be.
- No new symbols or operators are required to implement this feature.
- No need for special-casing async solutions in the grammar.
- Backwards compatibility.

## Other Examples

```js
//
// Define const variables while invoking function only once.
//
const(a) lastItem = getArray(), a[a.length-1];

const($$) neighbors =
  array.indexOf('abc'),
  array.slice($$ - 1, $$ + 2);

//
// Usage with closure to avoid re-calculating on each iteration.
//
let(_) topRankC =
  calculateRankC(),
  items.filter( item => item.score < _ ),
  _.sort(byScore),
  _[0];

//
// Temporary Bindings can also be used as an expression,
// allowing them to be used in return values.
//
const doubleShout = (str) =>
  let(x) = str.toUpperCase(), x+'!', x+' '+x;
```

## ES5 Conversion

```js
const($) x = 10, $ + 2, $ * 2;
//=>
// let $ be a unique variable within this scope
var $=10, x = ($= $ + 2, $= $ * 2);
```
