---
section: field-types
title: Relationship
---

# Field Type: `Relationship`

## Nested Mutations

Let's say you've got these lists:

```javascript
keystone.createList('User', {
  fields: {
    name: { type: Text },
    posts: { type: Relationship, ref: 'Post', many: true },
    company: { type: Relationship, ref: 'Org' },
  },
});

keystone.createList('Post', {
  fields: {
    title: { type: Text },
  },
});

keystone.createList('Org', {
  fields: {
    name: { type: Text },
  },
});
```

The to-many (`User.posts`) and to-single (`User.company`) relationships can be
mutated as part of a mutation on items in the parent list (eg; during a
`createUser`, or `updateUser` mutation, etc).

The available nested mutations:

<table>
  <tr>
    <th>Nested Mutation</th>
    <th>to-single relationship</th>
    <th>to-many relationship</th>
  </tr>
  <tr>
    <td><code>create</code></td>
    <td>
      Create a new item, and set it as the relation.<br />
      <i>Note: the previously set item (if any) is <b>not</b> deleted.</i>
    </td>
    <td>Create 1 or more new items, and append them to the list of related items.</td>
  </tr>
  <tr>
    <td><code>connect</code></td>
    <td>
      Filter for an item, and set it as the relation.<br />
      <i>Note: the previously set item (if any) is <b>not</b> deleted.</i>
    </td>
    <td>Filter for one or more items, and append them to the list of related items.</td>
  </tr>
  <tr>
    <td><code>disconnect</code></td>
    <td>
      Unset the relation (if any) if it matches the given filter.<br />
      <i>Note: the previously set item (if any) is <b>not</b> deleted.</i>
    </td>
    <td>
      Filter for one or more items, and unset them from the list of related items (if any).
      <br /><i>Note: the previously set items (if any) are <b>not</b> deleted.</i>
    </td>
  </tr>
  <tr>
    <td><code>disconnectAll</code></td>
    <td>
      Unset the relation (if any).<br />
      <i>Note: the previously set item (if any) is <b>not</b> deleted.</i>
    </td>
    <td>
      Unset the list of related items (if any).<br />
      <i>Note: the previously set items (if any) are <b>not</b> deleted.</i>
    </td>
  </tr>
</table>

### Order of execution

Nested mutations are executed in the following order:

1. `disconnectAll`
1. `disconnect`
1. `create`
1. `connect`

### Examples

#### Create and append a related item

Use the `create` nested mutation to create and append an item to a to-many
relationship:

<!-- prettier-ignore -->
```graphql
# Replace the company of a given User
query replaceAllPosts {
  updateUser(
    where: { id: "abc123" },
    data: {
      posts: {
        create: { title: "Hello World" },
      }
    }
  ) {
    # Now has a new post appended with the title "Hello World"
    posts {
      id
    }
  }
}
```

#### Append an existing item

Use the `connect` nested mutation to append an existing item to a to-many
relationship:

<!-- prettier-ignore -->
```graphql
# Replace the company of a given User
query replaceAllPosts {
  updateUser(
    where: { id: "abc123" },
    data: {
      posts: {
        connect: { id: "def345" },
      }
    }
  ) {
    # Now has an existing post appended with the id "def345"
    posts {
      id
    }
  }
}
```

#### Overriding a to-single relationship

Using either `create` or `connect` nested mutations is sufficient to override
the value of a to-single relationship (it's not necessary to use `disconnectAll`
as is the case for [to-many relationships](#overriding-a-to-many-relationship)):

<!-- prettier-ignore -->
```graphql
# Replace the company of a given User
query replaceAllPosts {
  updateUser(
    where: { id: "abc123" },
    data: {
      company: {
        connect: { id: "def345" },
      }
    }
  ) {
    # Will now be the company with id "def345"
    company {
      id
    }
  }
}
```

#### Overriding a to-many relationship

To completely replace the related items in a to-many list, you can perform a
`disconnectAll` nested mutation followed by a `create` or `connect` nested
mutation (thanks to the [order of execution](#order-of-execution)):

<!-- prettier-ignore -->
```graphql
# Replace all posts related to a given User
query replaceAllPosts {
  updateUser(
    where: { id: "abc123" },
    data: {
      posts: {
        disconnectAll: true,
        connect: [{ id: "def345" }, { id: "xyz789" }],
      }
    }
  ) {
    # Will now only contain posts with ids "def345" & "xyz789"
    posts {
      id
    }
  }
}

```