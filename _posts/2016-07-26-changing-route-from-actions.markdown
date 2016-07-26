---
title: changing route from actions
layout: post
categories: [Redux, React, React-router]
tags: [Redux, React, React-router]
published: True
---

````
# anywhere
import { hashHistory } from 'react-router';
hashHistory.push("/forgot_password")
````

or

````
// action
dispatch({
        type: K.ABC,
        payload: "boo"
      });

// reducer
const abc = (state=null, action) => {
  case K.ABC
    return action.payload
  default:
    return null;
}

// component
componentWillReceiveProps(nextProps){
    if(nextProps.abc){
      this.context.router.push("/forgot_password");
    }
  };
````