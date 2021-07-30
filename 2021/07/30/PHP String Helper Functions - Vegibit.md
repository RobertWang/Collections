> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [vegibit.com](https://vegibit.com/php-string-helper-functions/)

> PHP is incredibly powerful right out of the box, even if you aren't using any frameworks. We need to ......

PHP String Helper Functions
===========================

PHP is incredibly powerful right out of the box, even if you aren’t using any frameworks. We need to be fluent with the language itself as well as the frameworks we might like to use. Using the built in functions of the language we can create our own functions, or wrapper functions, of the built in ones. Sometimes this helps with just being able to more closely match your desired workflow. In this example we are going to create some String Helper functions to use that will help us to quickly work on strings in a very easy way. We’ll make use of 6 built in [PHP functions](https://vegibit.com/what-are-functions-in-php/ "What are Functions in PHP"), those being, [strtolower](https://vegibit.com/the-top-9-most-popular-php-string-functions/#strtolower "strtolower"), [strpos](https://vegibit.com/the-top-9-most-popular-php-string-functions/#strpos "strpos"), [substr](https://vegibit.com/the-top-9-most-popular-php-string-functions/#substr "substr"), [strlen](https://vegibit.com/the-top-9-most-popular-php-string-functions/#strlen "strlen"), preg_match_all, and [str_replace](https://vegibit.com/the-top-9-most-popular-php-string-functions/#strreplace "strreplace"), to build 4 new string helper functions. Let’s do it!

### Initial Configuration

Let’s first define some constants so that we can use these values in the helper functions.

```
// Include or exclude the setpoint
define ( "EXCL", true );
define ( "INCL", false );
// Get text before or after the setpoint
define ( "BEFORE", true );
define ( "AFTER", false );
```

### split_string()

First up is the **split_string()** function. It allows us to break a string into two pieces based on a setpoint, and then grab either side of the two pieces with or without the setpoint. It uses the built in PHP functions `strtolower`, `strpos`, and `substr`.

```
// Splits a string on a given setpoint, then returns what is before
// or after the setpoint. You can include or exclude the setpoint.
function split_string($string, $setpoint, $beforaft, $incorexc) {
	$lowercasestring = strtolower ( $string );
	$marker = strtolower ( $setpoint );

	if ($beforaft == BEFORE) {  // Return text BEFORE the setpoint
		if ($incorexc == EXCL) {
			// Return text without the setpoint
			$split_here = strpos ( $lowercasestring, $marker );
		} else {
			// Return text and include the setpoint
			$split_here = strpos ( $lowercasestring, $marker ) + strlen ( $marker );
		}
		$result_string = substr ( $string, 0, $split_here );
	} else {  // Return text AFTER the setpoint
		if ($incorexc == EXCL) {
			// Return text without the setpoint
			$split_here = strpos ( $lowercasestring, $marker ) + strlen ( $marker );
		} else {
			// Return text and include the setpoint
			$split_here = strpos ( $lowercasestring, $marker );
		}
		$result_string = substr ( $string, $split_here, strlen ( $string ) );
	}
	return $result_string;
}
```

### find_between()

Next up is the **find_between()** helper function. This handy guy basically just makes use of the **split_string()** function to help us to find a substring that is between a start and end point that we give it.

```
// Finds a string between a given start and end point. You can include
// or exclude the start and end point
function find_between($string, $start, $stop, $incorexc) {
	$temp = split_string ( $string, $start, AFTER, $incorexc );
	return split_string ( $temp, $stop, BEFORE, $incorexc );
}
```

### find_all()

Now we come to the **find_all()** function which is very useful. It makes use of the very popular `preg_match_all()` function to find all occurrences of a pattern in between a starting and ending delimiter. The real meat of this one is the pattern that gets passed into `preg_match_all()`. We can see the first argument is the pattern consisting of `"($start(.*)$end)siU"`. This takes our start point, captures anything and everything multiple times(`.*`), denoted by the dot and asterisk operators, stops at the end point, performs a case insensitive match (`i`), excludes line breaks (`s`), and is not greedy (`U`). These modifiers are extremely important for the RegEx to work properly.

```
// Uses a regular expression to find everything between a start
// and end point.
function find_all($string, $start, $end) {
	preg_match_all ( "($start(.*)$end)siU", $string, $matching_data );
	return $matching_data [0];
}
```

### delete()

The **delete()** function is useful for removing one, or many substrings from within a string. We simply pass it a string, a start point, and an end point, and the function will take care of the rest for us!

```
// Uses str_replace to remove any unwanted substrings in a string
// Includes the start and end
function delete($string, $start, $end) {
	// Get array of things that should be deleted from the input string
	$delete_array = find_all ( $string, $start, $end );
	
	// delete each occurrence of each array element from string;
	for($i = 0; $i < count ( $delete_array ); $i ++)
		$string = str_replace ( $delete_array, "", $string );
	
	return $string;
}
```

You may also like our lesson on setting up [testing helper functions](https://vegibit.com/laravel-testing-helpers/).

* * *

Using Our New Helper Functions
------------------------------

Ok, we now have our helper functions defined, but how do we use them? Well, we can create a file called `Stringhelpers.php` and dump all of the code into it just like this:

```
<?php
// string helper functions

// Include or exclude the setpoint
define ( "EXCL", true );
define ( "INCL", false );
// Get text before or after the setpoint
define ( "BEFORE", true );
define ( "AFTER", false );

// Splits a string on a given setpoint, then returns what is before
// or after the setpoint. You can include or exclude the setpoint.
function split_string($string, $setpoint, $beforaft, $incorexc) {
	$lowercasestring = strtolower ( $string );
	$marker = strtolower ( $setpoint );
	
	if ($beforaft == BEFORE) {  // Return text BEFORE the setpoint
		if ($incorexc == EXCL) {
			// Return text without the setpoint
			$split_here = strpos ( $lowercasestring, $marker );
		} else {
			// Return text and include the setpoint
			$split_here = strpos ( $lowercasestring, $marker ) + strlen ( $marker );
		}
		$result_string = substr ( $string, 0, $split_here );
	} else {  // Return text AFTER the setpoint
		if ($incorexc == EXCL) {
			// Return text without the setpoint
			$split_here = strpos ( $lowercasestring, $marker ) + strlen ( $marker );
		} else {
			// Return text and include the setpoint
			$split_here = strpos ( $lowercasestring, $marker );
		}
		$result_string = substr ( $string, $split_here, strlen ( $string ) );
	}
	return $result_string;
}

// Finds a string between a given start and end point. You can include
// or exclude the start and end point
function find_between($string, $start, $stop, $incorexc) {
	$temp = split_string ( $string, $start, AFTER, $incorexc );
	return split_string ( $temp, $stop, BEFORE, $incorexc );
}

// Uses a regular expression to find everything between a start
// and end point.
function find_all($string, $start, $end) {
	preg_match_all ( "($start(.*)$end)siU", $string, $matching_data );
	return $matching_data [0];
}

// Uses str_replace to remove any unwanted substrings in a string
// Includes the start and end
function delete($string, $start, $end) {
	// Get array of things that should be deleted from the input string
	$delete_array = find_all ( $string, $start, $end );
	
	// delete each occurrence of each array element from string;
	for($i = 0; $i < count ( $delete_array ); $i ++)
		$string = str_replace ( $delete_array, "", $string );
	
	return $string;
}
```

Now when we want to use any of these functions we can simply include this file like so:

```
include('Stringhelpers.php');
```

and we will have access to our new functions! Let’s test it out. We’ll create a new file called `stringtest.php`, include our helper file, and just call each new function on a small piece of text. Let’s see the results:

```
<?php
include('Stringhelpers.php');

$string = 'One really awesome string';

echo split_string($string, 'awesome', BEFORE, INCL);
//  One really awesome

echo split_string($string, 'awesome', AFTER, EXCL);
//  string


echo find_between($string, 'One', 'string', EXCL);
//  really awesome

echo find_between($string, 'really', 'string', INCL);
//  really awesome string


$string = 'One really awesome string. Two really awesome strings!';
print_r(find_all($string, 'really', 'string'));
//  Array ( [0] => really awesome string [1] => really awesome string ) 


$string = 'Why do silly, green, or yellow frogs jump? Why do silly, green, or yellow frogs jump?';

echo delete($string, 'silly', 'yellow');
//  Why do frogs jump? Why do frogs jump?
?>
```

**Wow!** That was really cool! Now what’s the big deal with all this Laravel we’ve been talking about, then shifting gears and creating some simple PHP string helper functions?! Listen now Grasshopper, there is a method to our madness! You see before frameworks, this is how we did things. We created functions, we put them in files, we included those files into other files, and tada!, WordPress was born Just kidding – but in all seriousness, we all probably have a lot of great functions and snippets of code in our arsenal. Once we start using a great framework like Laravel, do we have to abandon those old functions that we’ve come to know and love? But of course not! Now that we have created a file containing a handful of helper functions, we will have something to work with for our next tutorial. In our next episode, we’ll take a look at how to convert our old file based functions, and put them into a dedicated class which can be added to a libraries folder in Laravel. Then we’ll set up autoloading of any libraries we want to add to our project, and we’ll be able to use the new hotness of Laravel, along with any of our own Library functions we’ve collected over the years!

Thank you for reading **PHP String Helper Functions** – **Please do share using the buttons below!**

#[php](https://vegibit.com/tag/php/)