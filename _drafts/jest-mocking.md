## Thoughts on Mocking

- Place a new file with the same name of the file you want to mock in the __mocks__ directory next to the file you want to mock. When you do `jest.mock('./path/to/file')` it will load the mocked file instead.
- An alternative to the same is to provide a factory function as 2nd param like `jest.mock('./path/to/file', () => { return {a: 'foo'} })`. Now whatever the factory function returns is what will get loaded when the mocked file is required by anyone.
- these are not best when mocking a big file with lots of things in it, which call each other, as we will have to do write mocked functions for all of them in the mocked file.
- for mocking node modules create a __mocks__ directory next to it. Also no need to call `jest.mock` for them. So to mock `fs` we create a file called fs.js at __mocks__.
- When a mock file for a node package is placed, then we again we must call `jest.mock`. So to mock `lodash` we create a file called lodash.js at __mocks__.
- .
  ├── config
  ├── __mocks__
  │   └── fs.js
  │   └── lodash.js
  ├── models
  │   ├── __mocks__
  │   │   └── user.js
  │   └── user.js
  ├── node_modules
  │   └── lodash
  │     └── src
  │         └── index.js
  └── views
- `jest.genMockFromModule` this takes a module, generates its mocked version and then gives it right back to you. this way you can then extend it, alter it etc. and then further return your mock. This is something you would do in the mocked file you are creating.
- `jest.mock` also takes a module, generates its mocked version and gives it back when and where it is being required. it also takes a factory method to run and provide results instead of the automocked version. This will also pick and use the sample mocked file placed in __mocks__ directory. This is something you would use in the test file before loading the subject, so when the subject is called it internally will get the mocked verion of the file.
- another thing that can happen is that you do `jest.mock` in the test file and then require it right there. now you are getting the automocked version right in your test file. then you modify it. then your tests run and the load the subject which internally loads the module and get the modified automocked file.
```
jest.mock('../../services/getKinesisVideoObject')
const kinesisVideo = require('../../services/getKinesisVideoObject')
kinesisVideo.getDataEndpoint.mockImplementation(() => { return {
  promise: () => Promise.resolve({DataEndpoint: 'http://abc.com'})
}})
test('calls getDataEndpoint', async () => {
    const streamUrl = await getStreamingSessionUrl()
    expect(kinesisVideo.getDataEndpoint).toHaveBeenCalled()
  })
``` 
- `require.actual` takes the module and returns it as is, and not the automocked version. used in factory method of jest.mock if you want everything original with some changes.
- mocking es6 classes works, just that the manual mock you define needs to also be either an es6 class or a function constructor.
- want to over-ride a instance method which is on a class. like 
```
c = new Calculator()
c.addition // i want to over-ride this
```
it can be done by using prototype chain as we know that the method is here `Calculator.prototype.addition`  
```
jest.mock('./Calculator')
c = require('./Calculator') // this gets us an automocked version of calculator
c.prototype.addition.mockReturnValue('1000')
```
```
jest.mock('../calculator', () => {
  return jest.fn(() => {
    return { addition: jest.fn().mockReturnValue(100) }
  })
})
const result = sum(1,2)
expect(result).toBe(100)
```

```
const myAddition = jest.fn().mockReturnValue(123)
jest.mock('../calculator', () => {
  return jest.fn(() => {
    return { addition: myAddition }
  })
})
const result = sum(1,2)
// ReferenceError
// The module factory of `jest.mock()` is not allowed to reference any out-of-scope variables
expect(result).toBe(100) 
```
```
// see the name, it starts the mocked
const mockedAddition = jest.fn().mockReturnValue(100)
jest.mock('../calculator', () => {
  return jest.fn(() => {
    return { addition: mockedAddition }
  })
})
const result = sum(1,2)
expect(result).toBe(100)
expect(mockedAddition).toHaveBeenCalled()
```
- remember that calling `new` on javascript makes the function and act as a constructor but if this function is returning a value, then the same value is returned regardless of new. 
```
const foo = function() {
  const obj = {
    name() { return "sandeep" }
  }
  return obj
}

console.log(foo().name()) // sandeep
console.log((new foo()).name()) // sandeep
```
- the above fact works in our advantage when creating a mock of a constructor which returns a new value. We can use `jest.fn` to create the mock and return a value. This value will now be returned whether `new` is used or not
```
const MyMockedFunction = jest.fn().mockImplementation(() => {
  return { name: function() { return 'sandeep' } }
})

console.log(MyMockedFunction().name()) // sandeep
console.log((new MyMockedFunction()).name()) // sandeep

```
- jest.fn(SomeFunctionHere) vs jest.fn().mockReturnValue vs jest.fn().mockImplementation
- Gotcha
The mocked function which you kept outside so you can test whether it was called or not, should be placed above the require statement to the subject file, otherwise you will get reference error. Remeber that your jest.mock call is being hoiseted to the top, but thats about it.
```
const foo = require('./foo')
// Cannot access 'mockedFb' before initialization
const mockedFn = jest.fn() // Reference error because this line should be above the require to Foo
jest.mock('./foo', () => { // placement of this is taken care of for you
    return { bla: mockedFn }
)
```
Correct way:
```
const mockedFn = jest.fn()
jest.mock('./foo', () => {
    return { bla: mockedFn }
)
const foo = require('./foo')
```
- if you have a mock and you are doing multiple assertions, in each assertion you are getting the first call to it and checking its params etc. this is normal. just make sure that between assertions you clear the history of the mock by doing `myMock.mockClear()`. this gets rid of the `myMock.mock` object therefore resetting `myMock.mock.calls` etc.
- you can go a step further and do `myMock.mockReset()`. this does what `mockClear` and also gets rid of of the mock implementation and mock return values 
