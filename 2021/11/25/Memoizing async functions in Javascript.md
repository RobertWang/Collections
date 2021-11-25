> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [stackfull.dev](https://stackfull.dev/memoizing-async-functions-in-javascript)

> Memoization is a useful concept. It helps avoid time taking or expensive calculations after it's been......

Memoization is a useful concept. It helps avoid time taking or expensive calculations after it's been done once. Applying memoization to a synchronous function is relatively straightforward. This article aims to introduce the overall concept behind memoization and then dive into the problems and their solutions while trying to memoize async functions.

Memoization
-----------

Let's start with memoizing a pure function. Let's say we have a function called `getSquare`, which returns the square of the given:

```
function getSquare(x){
     return x * x
  }
```

To memoize this we can do something like this:

```
const memo = {}

function getSquare(x){
    if(memo.hasOwnProperty(x)) {
      return memo[x]
    }
    memo[x] = x * x
    return memo[x]
}
```

So, with few lines of code, we've memoized our `getSquare` function.

Let's create a `memoize` helper. It would accept a pure function as the first argument and a`getKey` function (which returns a unique key given argument of the function) as the second argument, to return a memoized version of the function:

```
function memoize(fn, getKey){
  const memo = {}
  return function memoized(...args){
     const key = getKey(...args)
     if(memo.hasOwnProperty(key)) return memo[key]

     memo[key] = fn.apply(this, args)
     return memo[key]
  }
}
```

We can apply this function to `getSquare` as follows:

```
const memoGetSquare = memoize(getSquare, num => num)
```

Memoizing a function accepting multiple arguments:

```
const getDivision = (a, b) => a/b


const memoGetDivision= memoize(getDivision, (a, b) => `${a}_${b}`)
```

Memoizing async functions
-------------------------

Let's say there' a function called `expensiveOperation(key)` which accepts a key as an argument and performs some async operation before returning the final result via a callback:

```
expensiveOperation(key, ( data) => {
   
})
```

Let's use similar notion as above to memoize this function:

```
const memo = {}

function memoExpensiveOperation(key, callback){
  if(memo.hasOwnProperty(key)){
    callback(memo[key])
    return
  } 

  expensiveOperation(key, data => {
   memo[key] = data
   callback(data)
  })
}
```

So that was pretty easy. But wait! It doesn't solve the whole problem yet. Consider the following scenario:

1- Invoked `expensiveOperation` with key 'a'

2- While #1 is still in progress, invoked it again with same key

The function would run twice for the same operation because #1 is yet to save the final data in `memo`. That's not something we wanted. We would instead want concurrent calls to be resolved at once after the earliest call is complete.

To address this issue we can track the operations currently in progress, say we do so by putting it in some sort of queue. Now, if we receive a call for an operation, while it's still in queue, we further enqueue the new call. This way we keep accumulating the repetitive calls and once the operation is done, all of these are processed in one go.

```
const memo = {}, progressQueues = {}

function memoExpensiveOperation(key, callback){
     if(memo.hasOwnProperty(key)){
       callback(memo[key])
       return
      }

     if(!progressQueues.hasOwnProperty(key)){
        
        progressQueues[key] = [callback]

      } else {
       
        progressQueues[key].push(callback);
        return
      }

      expensiveOperation(key, (data) => {
           
           memo[key] = data 
           
           for(let callback of progressQueues[key]) {
                callback(data)
           }
           
           delete progressQueue[key]
       })

}
```

We can go a step further, just like the last section, and create a re-usable helper say `memoizeAsync`:

```
function memoizeAsync(fn, getKey){
   const memo = {}, progressQueues = {}

   return function memoized(...allArgs){
       const callback = allArgs[allArgs.length-1]
       const args = allArgs.slice(0, -1)
       const key = getKey(...args)

        if(memo.hasOwnProperty(key)){
            callback(key)
            return
        }


        if( !progressQueues.hasOwnProperty(key) ){
           
           progressQueues[key] = [callback]
        } else {
           
           progressQueues[key].push(callback);
           return
        }

        fn.call(this, ...args , (data) => {
           
           memo[key] = data 
           
           for(let callback of progressQueues[key]) {
                callback(data)
           }
           
           delete progressQueue[key]
       })
   }
}



const memoExpensiveOperation = memoizeAsync(expensiveOperation, key => key)
```

Promises
--------

Let's say we have a function `processData(key)` which accepts a key as argument and returns a Promise. Let's see how it can be memoized.

### Memoizing the underlying promise:

Simplest way would be to memoize the promise issued against the key. Here's how it would look like:

```
const memo = {}
function memoProcessData(key){
  if(memo.hasOwnProperty(key)) {
    return memo[key]
  }

  memo[key] = processData(key) 
  return memo[key]
}
```

The code is fairly simple and self-explanatory here. We can use the `memoize` helper we created a while ago:

```
const memoProcessData = memoize(processData, key => key)
```

### Can we memoize the value returned by Promise?

Yes. We can apply the same approach as the callback here. Though it might be an overkill for the sake of memoizing such a function:

```
const memo = {},  progressQueues = {}

  function memoProcessData(key){

    return new Promise((resolve, reject) => {
      
      if(memo.hasOwnProperty(key)){
        resolve(memo[key])
        return;
      }

      if( !progressQueues.hasOwnProperty(key) ){
        
        progressQueues[key] = [[resolve, reject]]

      } else {
       
        progressQueues[key].push([resolve, reject]);
        return;
      }


      processData(key)
        .then(data => {
            memo[key] = data; 
            
            for(let [resolver, ] of progressQueues[key])
              resolver(data)
        })
        .catch(error => {
           
           for(let [, rejector] of progressQueues[key])
              rejector(error);
         })
        .finally(() => {
          
           delete progressQueues[key]
         })
    })
  }
```

Further improvement
-------------------

Since we're using a `memo` object to keep track of memoized operations, with too many calls to `expensiveOperation` with various keys(and each operation returning a sizeable chunk of data after processing) the size of this object may grow beyond what's ideal. To handle this scenario we can use a cache eviction policy such as [LRU](https://en.wikipedia.org/wiki/LRU) (Least Recently Used). It would ensure we're memoizing without crossing memory limits!