---
title: Writing maintainable tests using the data driven approach
date: 2016-06-07 06:49:08
tags:
- Testing
- Maintainability
---

One of the main considerations while writing automated tests is to make them more mainatainable. Tests shouldn't get in the way of refactoring the source. This can be achieved by making the tests [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) and abstracting out repeating logic.

Certain logic like form validation, url parsing etc make tests unmaintainable by repeating the same tests again and again with different inputs and outputs.

Consider a simple example of validating username:

```javascript
// validation.js

function isValidUserName (userName) {
	if (userName.isBlank ||
		userName.isLessThanTwoCharacters ||
		userName.hasSpecialCharacters ||
		userName.hasNumbers) {
		return false;
	} else {
		return true;
	}
}

```

The tests for the above function would be something like this:

```javascript
// validation.test.js

it('should reject blank input', function () {
	var isValid = isValidUserName('');
	expect(isValid).to.be.false;
});

it('should reject input with less than two characters', function () {
	var isValid = isValidUserName('a');
	expect(isValid).to.be.false;
});

it('should reject input with special characters', function () {
	var isValid = isValidUserName('abc#');
	expect(isValid).to.be.false;
});

it('should reject input with numbers', function () {
	var isValid = isValidUserName('ab3c');
	expect(isValid).to.be.false;
});

```

These kind of tests would be tedious to maintain for certain kind of logic.

A better way would be to use the [data driven approach](https://en.wikipedia.org/wiki/Data-driven_testing). In this approach, we decouple the data and the actual testing code.

 ```javascript
 // validation.data.json

 {
 	"username": [
 		{
 			"title": "should reject blank input",
 			"input": "",
 			"isValid": false
 		},
 		{
 			"title": "should reject input with less than two characters",
 			"input": "a",
 			"isValid": false
 		},
 		{
 			"title": "should reject input with special characters",
 			"input": "abc#",
 			"isValid": false
 		},
 		{
 			"title": "should reject input with numbers",
 			"input": "ab3c",
 			"isValid": false
 		}
 	]
 }
 ```
And modify our test file as

```javascript
// validation.test.js

var data = require('validation.data');

data.username.forEach(function (test) {
	it(test.title, function () {
		var isValid = isValidUserName(test.input);
		expect(isValid).to.equal(test.isValid);
	});
});

```

This test can now be quickly refactored if needed. A nice bonus of using this approach is that by extracting the test data from actual test cases, the test scenarios can be understood in a glance.
