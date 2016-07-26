---
title: Using Nock For Testing Ajax Requests
layout: post
categories: [mocha, nock, react, redux, testing]
tags: [mocha, nock, react, redux, testing]
published: True
---

https://github.com/node-nock/nock

1. Update Url to be a full url so it can be reached from Mocha as webpack dev server proxy wont work

````
export const getAllPhoneNumbers2 = (pageNumber) => {
  return axios.get(`http://3000/phone_numbers.json`, {
    params:{
      page: pageNumber
    }
  });
}

````

2. In the test before the action is called whick makes ajax add

````
nock.recorder.rec();
````

3. Turn on backen server & run the tests. Check console for code snippet. It will print out everything needed to reproduce the AJAX call next time without making HTTP requests.

````
nock(/http:\/\/localhost/, {"encodedQueryParams":true})
      .post('/users/sign_in', {"user":{"email":"sandeep45@gmail.com","password":"12345678"}})
      .reply(201, {"authentication_token":"WAGzPbE49sh9cSD21mea","created_at":"2016-07-23T18:41:43Z","email":"sandeep45@gmail.com","id":1,"updated_at":"2016-07-25T01:53:49Z"}, { location: 'http://localhost:3000/',
      'content-type': 'application/json; charset=utf-8',
      'x-ua-compatible': 'IE=Edge',
      etag: '"164f0a8fa5b7d667ae5a55bac5da7633"',
      'cache-control': 'max-age=0, private, must-revalidate',
      'x-request-id': '37d427e3ae2427db92bcc66db6fd1033',
      'x-runtime': '0.067432',
      server: 'WEBrick/1.3.1 (Ruby/2.1.3/2014-09-19)',
      date: 'Mon, 25 Jul 2016 01:53:49 GMT',
      'content-length': '156',
      connection: 'close',
      'set-cookie': [ '_meme_backend_session=; expires=Thu, 01-Jan-1970 00:00:00 GMT' ] });
````

4. Change thr domain name to not be tied to any port so it works after the port has been removed.

5. Remove the `nock.recorder.rec();` and add the above snippet. Remove the port from ajax request.

6. Now you can run your tests again and it will work just like before without needing to call the backend.

7. You can seee this in action here : https://github.com/sandeep45/redux-template

8. Pro-Tip: Move all snippets required to nock to seperate modules and import as needed.