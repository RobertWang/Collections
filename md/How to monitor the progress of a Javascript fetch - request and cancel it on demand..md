> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.q-bit.me](https://blog.q-bit.me/monitoring-and-canceling-a-javascript-fetch-request/)

> Waiting blows. That goes for many situations in life, including when a website you visit is busy load......

TL: DR -> Take me to the code: [https://github.com/tq-bit/fetch-progress](https://github.com/tq-bit/fetch-progress)

[In an earlier post,](https://blog.q-bit.me/make-api-requests-with-javascript/) I've already given an overview of how to interact with an API using fetch. In this article, I'd like to dig deeper into two more detailed use-cases:

*   Monitor the download progress while making an HTTP request.
*   Gracefully cancel a request by a user's input.

If you would like to follow along, you can use this Github branch to get started. It includes no Javascript, just some styles and HTML: [https://github.com/tq-bit/fetch-progress/tree/get-started](https://github.com/tq-bit/fetch-progress/tree/get-started).

![](data:image/png;base64,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)This is the UI we will start off with. The progress indicator will visualize the fetch - progress 

So spin up your favorite code editor and let's dive in.

Create the basic fetch request
------------------------------

Before starting with the advanced stuff, let's build up a simple function. The task is to develop a piece of utility code that allows you to search for universities. Fortunately, [Hipo](https://github.com/Hipo) has just the tool to build up upon.

*   I'm using [this repository](https://github.com/Hipo/university-domains-list)'s hosted API as a starting place.
*   Its root URL is [http://universities.hipolabs.com/](http://universities.hipolabs.com/).
*   I'd like to restrict my search to all universities in the USA with a query.
*   On the technical side, I'd like to keep my fetch logic inside a wrapper function.

That being said, let's start by adding the following code to the `client.js` file:

```
export default function http(rootUrl) {
  let loading = false;

  let chunks = [];
  let results = null;
  let error = null;


  

  const json = async (path, options,) => {
    loading = true

    try {
      const response = await fetch(rootUrl + path, { ...options });

      if (response.status >= 200 && response.status < 300) {
        results = await response.json();
        return results
      } else {
        throw new Error(response.statusText)
      }
    } catch (err) {
      error = err
      results = null
      return error
    } finally {
      loading = false
    }
  }

  return { json }
}


```

```
import http from './client.js';
const { json } = http('http://universities.hipolabs.com/');


const progressbutton = document.getElementById('fetch-button');


progressbutton.addEventListener('click', async () => {
  const universities = await json('search?country=United+States');
  console.log(universities);
});


```

```
export default function http(rootUrl) { 

  
  const _readBody = async (response) => {
    const reader = response.body.getReader();
    
    
    let received = 0;

    
    while (loading) {
      const { done, value } = await reader.read();
      if (done) {
        
        loading = false;
      } else {
        
        chunks.push(value);
      }
    }

    
    let body = new Uint8Array(received);
    let position = 0;

    
    for (let chunk of chunks) {
      body.set(chunk, position);
      position += chunk.length;
    }

    
    return new TextDecoder('utf-8').decode(body);
  }
  return { json }
}


```

```
  
  
  results = await _readBody(response)
  return JSON.parse(results)


```

```
  const _readBody = async (response) => {
    const reader = response.body.getReader();
    
    
    const length = +response.headers.get('content-length'); 
    
    
    let received = 0; 
  
  if (done) {
      
      loading = false;
    } else {
      
      chunks.push(value);
      
      
      received += value.length; 
    }
  }


```

```
  const _readBody = async (response) => {
    
    
    
    while (loading) {
      const { done, value } = await reader.read();
      const payload = { detail: { received, length, loading } }
      const onProgress = new CustomEvent('fetch-progress', payload);
      const onFinished = new CustomEvent('fetch-finished', payload)

      if (done) {
        
        loading = false;
        
        
        window.dispatchEvent(onFinished)
      } else {
        
        chunks.push(value);
        received += value.length;
        
        
        window.dispatchEvent(onProgress); 
      }
    }
    
  }


```

```
import http from './client.js';


const progressbar = document.getElementById('progress-bar');
const progressbutton = document.getElementById('fetch-button');
const progresslabel = document.getElementById('progress-label');
const { json } = http('http://universities.hipolabs.com/');

const setProgressbarValue = (payload) => {
  const { received, length, loading } = payload;
  const value = ((received / length) * 100).toFixed(2);
  progresslabel.textContent = `Download progress: ${value}%`;
  progressbar.value = value;
};


progressbutton.addEventListener('click', async () => {
  const universities = await json('search?country=United+States');
  console.log(universities);
});

window.addEventListener('fetch-progress', (e) => {
  setProgressbarValue(e.detail);
});

window.addEventListener('fetch-finished', (e) => {
  setProgressbarValue(e.detail);
});


```

```
const _resetLocals = () => {
  loading = false;

  chunks = [];
  results = null;
  error = null;

  controller = new AbortController();
}


```

```
export default function http(rootUrl) {
  let loading = false;

  let chunks = [];
  let results = null;
  let error = null;

  let controller = null; 
  const json = async (path, options,) => {
    _resetLocals();
    loading = true
  
  }



```

```
const json = async (path, options,) => {
  _resetLocals();
  let signal = controller.signal; 
  loading = true

  try {
    const response = await fetch(rootUrl + path, { signal, ...options });
  
  }

}


const cancel = () => {
  _resetLocals();
  controller.abort();
};


return { json, cancel }


```

```
const abortbutton = document.getElementById('abort-button');
const { json, cancel } = http('http://universities.hipolabs.com/');


abortbutton.addEventListener('click', () => {
  cancel()
  alert('Request has been cancelled')
})

```

If you now hit **Fetch** and **Cancel** **Request** right after, you will see an alert indicating that the request, even if it returns an HTTP status of 200, returns no data.

![](https://blog.q-bit.me/content/images/2021/08/Screenshot-from-2021-08-16-17-22-52.png)

Update: Vue 3 composition function for fetch
--------------------------------------------

I have recreated this functionality with Vue 3's Composition API. If you are looking to implement monitoring and canceling fetch requests in your Vue app, you should have a look into this Gist:

[

A Vue 3 Compostion Hook (MVP) for the Fetch API. Its public methods can be used to monitor and cancel a fetch request.

A Vue 3 Compostion Hook (MVP) for the Fetch API. Its public methods can be used to monitor and cancel a fetch request. - Component.vue

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)262588213843476

](https://gist.github.com/tq-bit/79d6ab61727ebf29ed0ff9ddc4deedca)

What next?
----------

Unfortunately, by the time I researched for this article, I could not find a common way to monitor upload progress. The official whatwg Github repository has an [open issue](https://github.com/whatwg/fetch/issues/607) on a feature named `FetchObserver`. However, it seems we'll have to be patient for it to be implemented. Perhaps, it will make the features described in this article easier as well. The future will tell.

[

FetchObserver (for a single fetch) · Issue #607 · whatwg/fetch

In #447 (comment) @jakearchibald sketched some APIs based on @stuartpb&#39;s work which @bakulf then implemented: https://developer.mozilla.org/en-US/docs/Web/API/FetchObserver https://dxr.mozilla....

GitHubwhatwg

](https://github.com/whatwg/fetch/issues/607)