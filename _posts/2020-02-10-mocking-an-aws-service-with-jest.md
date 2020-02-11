---
title: Mocking An AWS Service With jest
status: True
layout: post
categories: [javascript, Jest, AWS]
tags: [javascript, Jest, AWS]
published: True
---

So we have a function which uses an AWS Service like this:

```
// fileName: foo.js
const AWS = require('aws-sdk')
const kinesisVideo = new AWS.KinesisVideo()

const getUrlBasedOnUserInput = () => {
    const dataEndPointResponse = await kinesisVideo.getDataEndpoint({
        StreamName: 'foo',
        APIName: 'GET_HLS_STREAMING_SESSION_URL'
      }).promise()
    return dataEndPointResponse.DataEndpoint
}
module.exports = getUrlBasedOnUserInput
```

Now we want to test it. In terms of AWS, we want to know that:
1. Our code called the `getDataEndpoint` function of AWS
2. It was called with the right params
3. It returns the results this functions promise function resolves with

Everything here is straightforward jest testing besides knowing that function `getDataEndpoint` is called on an instance of `AWS.KinesisVideo`

```
// fileName: foo.test.js
const mockedGetDataEndPoint = jest.fn(() => {
  return {
    promise: jest.fn(() => Promise.resolve({ DataEndpoint: 'http://foobar.com' }))
  }
})
jest.mock('aws-sdk', () => {
  return {
    KinesisVideo: jest.fn(() => {
      return { getDataEndpoint: mockedGetDataEndPoint }
    })
  }
})
const foo = require('../foo.js')
describe('foo', () => {
    test('it calls getDataEndpoint on AWS.KinesisVideo', async () => {
        const result = await foo()
        expect(mockedGetDataEndPoint).toHaveBeenCalled()        
        expect(result).toBe('http://foobar.com')
    })
})
```  

Here is whats going on here:

- First we mock out aws-sdk by doing `jest.mock('aws-sdk', () => {})` and provide a custom factory.
- In the factory we return a json which has `KinesisVideo` defined.
- Note that the subject is doing `new` on `AWS.KinesisVideo`. This means that its a constructor. So we define it as a function by doing `jest.fn`
- Now in javascript if you return something from a function, that thing will be returned regardless of whether its called with the `new` keyword or not. See example below.
```
function foo() {
    return { name: 'foobar' }
}
foo() // returns { name: 'foobar' }
new foo() // { name: 'foobar' }
```
- So we make this constructor function return a JSON with `getDataEndpoint` as a key and whose value is a mockedFunction we want to ensure is called. We do this because we know the subject is doing `new AWS.KinesisVideo().getDataEndpoint()`
- We keep this mockedFunction outside in a variable so we can reference it from our tests and ensure that it was called etc.
- In the implementation of this mockedFunction we return a Promise which resolves with data the subject is looking for.
- Finally note, that we name this mocked function starting with the words `mock` and initialize it before its used by placing it on the top.