---
title: Rails like scope methods in Objection.js (NodeJS ORM)
status: True
layout: post
categories: [Nodejs, ORM, Objectionjs]
tags: [Nodejs, ORM, Objectionjs]
published: True
---

Let say we have a model called `Label`

```
const { Model } = require('objection')
class Label extends Model {
    static get tableName() {
        return "labels"
    }
    static get jsonSchema () {
        return {
            type: 'object',
            required: [],
            properties: {
                id: { type: 'integer' },
                name: { type: 'string' }
            }
        }
    }
}
```

Now we want to get the last Label in the model.

```
const label = await Label.query().orderby('id', 'desc').limit(1).first()
```

Although this gets us the last label, it has a few shortcomings:

1. It is verbose
2. It requires too much repeated typing and thus prone to errors
3. Its harder to test
4  It doesn't read well
5. things only get worse when its used in conjunction with another method which gets all labels which start with "A".

Here are 3 ways to approach this:
1. Modifiers
2. A Regular Class Method
3. Custom QueryBuilder Object

Lets dive into each of these one-by-one. 

## Modifiers

Modifiers is my preferred way to solve this. We specify a function on the modifiers object which:
 1. receive the query as a param
 2. it then modifies the query by adding its filters etc.
 
```
Label.modifiers.last = query => {
    query.orderby('id', 'desc').limit(1).first()
}
```  

Now lets get the last record by using this modifier

```
const label = await Label.query().modify('last')
```

This reads so much better, encapsulates all logic under one function and we can test that one function easily.

The logs show that it ran:

```
select "labels".* from "labels" order by "id" DESC limit 1
```

### With params

Lets build another modifier which gets all labels which start with the passed in letters

```
Label.modifiers.startsWith = (query, letters) => {
    query.where('name', 'like', `${letters}%`)
}
```

Now lets run it

```
labels = await Label.query().modify('startsWith', 'XYYZ')
```

And logs show:

```
select "labels".* from "labels" where "name" like "AC%"
```

### Combining multiple modifier functions

This is where i think modifier functions start to shine, just like scopes do in Rails.
 
So lets say we need the last label which starts with 'A'. We can achieve this by using our `startsWith` & `last` modifier functions together.

```
const label = await Label.query().modify('startsWith','A').modify('last') 
```

And our logs have:

```
select "labels".* from "labels" where "name" like "A%" order by "id" DESC limit 1
```

## Class method on Label

A regular static method on label class. We can have this method return the last record:

```
Label.last = () => {
    return await Label.orderby('id', 'desc').limit(1).first()
}
```  

This gets the job done, but not as good as a modifier function. Yes it reads good and encapsulates the work but it doesn't return the query object and thus can't be chained

## Custom QueryBuilder

We can define a method on the `query()` object itself. This will allow us to modify the query by calling an internal method of the query object, without writing the words `modify` and explicitly making it clear that we are modifying the query.

Lets see an example:

```
class MyQueryBuilder extends QueryBuilder {
  last () {
    logger.info('inside last')
    this.orderBy('id', 'desc').limit(1).first()
    return this
  }
}

class Label exteds Model {
    static get QueryBuilder () {
        return MyQueryBuilder
    }
}
```

Now to use it:

```
cons label = await Label.query().last()
```

I think this approach is an abuse of power. It works, but we have a cleaner way of modifying the query and we should do that instead of defining a custom query object which has special internal methods. 

I think this custom query class might have good use cases though for other things like logging, making some other service calls etc.

## Conclusion

`modifiers` are great. the ability to chain them make them an asset.

## What's Next

Use modifiers with complex queries which use:
- join
- graphFetch (eager loading)
- use `ref` where we have ambiguous table names
