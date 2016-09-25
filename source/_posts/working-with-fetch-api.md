---
title: Working with Fetch api
keywords: JavaScript
date: 2016-09-25 18:20:00
tags:
- JavaScript
---

Fetch is a much needed improvement over XHR, it simplifies making network requests by exposing an easy to use api and having promise support out of the box.

```javascript
fetch(url).then(function (response) {
	return response.json();
});

```

### Wrapping fetch

While the above example is good enough for most cases, sometimes you might need to send the same headers in all the requests or handle all the  responses in the same way. Doing so in each and every `fetch` call would be duplicating a lot of code. This can solved by creating a wrapper around the fetch method and using that wrapper throughout the application instead of fetch.

```javascript
// fetch-wrapper.js

function fetchWrapper(url, options) {
	var options = options || {};
	options.headers['Custom-Header'] = 'Your custom header value here';

	return fetch(url, options);
}
```

```javascript
// books.js

fetchWrapper('/api/books')
	.then(function (data) {
		console.log(data);
	});
```

### Rejecting on HTTP errors

Coming from jQuery.ajax, one of the main gotcha's about fetch is that it does not reject on HTTP errors - It only rejects on network failures. While this makes sense because any response (whether 2xx or 4xx etc) is still a response and thereby a 'success', you might want fetch to reject on http errors so that the `catch` part of your promise chain can handle them appropriately.

```javascript
// fetch-wrapper.js

function fetchWrapper(url, options) {
	return fetch(url, options)
		.then(handleResponse);
}

function handleResponse (response) {
	if (response.ok) {
		return response.json();
	} else {
		throw new Error(response.statusText);
	}
}

```

```javascript
// books.js

fetchWrapper('/api/books')
	.then(function (data) {
		console.log(data);
	})
	.catch(function (error) {
		console.error(error);
	});
```

### Handling JSON responses

If all the responses are guaranteed to be JSON, then we can parse them before passing them down the promise chain. Since fetch throws TypeError on network errors, we can handle it in `handleNetworkError` to throw a JSON object similar to ones we get from our backend.

```javascript
// fetch-wrapper.js

function fetchWrapper(url, options) {
	return fetch(url, options)
		.then(handleResponse, handleNetworkError);
}

function handleResponse (response) {
	if (response.ok) {
		return response.json();
	} else {
		return response.json().then(function (error) {
			throw error;
		});
	}
}

function handleNetworkError (error) {
	throw {
		msg: error.message
	};
}

```

```javascript
// books.js

fetchWrapper('/api/books')
	.then(function (data) {
		console.log(data);
	})
	.catch(function (error) {
		console.error(error.msg);
	});
```

### Timeouts

There's no support for timeouts in the fetch api, though this can be achieved by creating a promise that rejects on timeout and using it with the Promise.race api.

```javascript
function timeout (value) {
	return new Promise(function (resolve, reject) {
		setTimeout(function () {
			reject(new Error('Sorry, request timed out.'));
		}, value);
	})
}

Promise.race([timeout(1000), fetch(url)])
	.then(function (response) {
		console.log(response);
	})
	.catch(function (error) {
		console.error(error);
	})

```

But keep in mind that since fetch has no support for aborting the request, the above example only rejects the promise but the request itself is still alive. This behavior is different from XHR based libraries which abort the request when it takes longer than the timeout value.

### Browser support

Fetch is [supported in most browsers](http://caniuse.com/#feat=fetch) and has a [polyfill](https://github.com/github/fetch) for those who don't support it.

### Limitations

Fetch is being developed iteratively and there are certain things that it does not support like [monitoring progress](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#Monitoring_progress), [aborting](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/abort) a request etc. If these are absolutely necessary to your application, then your should use XHR or its abstractions like jQuery.ajax, axios etc instead of fetch.

### Closing thoughts

Though it seems to be limited compared to XHR, I think the current feature set is good enough for most of the cases. The simple api makes it beginner friendly and (future) native support means one less dependency to load.
