---
layout: post
title:  "A simple React Router"
date:   2017-06-28 16:25:00 -0400
categories: react redux routing hacking
---

During our last React / Redux project, we started to take our existing single page, single component app and wanted to add some more URL based routing.  We had done some hacky stuff earlier in the project but had a need to make our URL parameter parsing real.  So, we took at look at some of the options out there.

We liked the react-router project because it is widely used and seemed like a good bridge between the URL and the react props.  But, since we were using redux, we had state stored in redux.  There are some projects designed to help the integration of react-router and redux, however, they seemed either hacky (because they never guaranteed when the redux store would be updated) or super complex (requiring lots of boiler plate).  We decided to create a few components to bridge the gap.

````javascript
var foo = 'bar';
````

