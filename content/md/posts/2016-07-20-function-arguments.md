{:layout :post, :title "\"Function Arguments\"", :date "2016-07-20 09:41:05 +1000"}
In [Clean Code](https://www.bookdepository.com/Clean-Code-Robert-C-Martin/9780132350884?a_aid=rickerbh) it's advocated that the ideal number of arguments for a function is 0. Then, 1, 2, and in very rare cases 3. No functions should take more than 3 arguments.

In general, I agree with this. Fewer function arguments = fewer separate parts to understand and should aid with readability (and understanding). There are a couple of complexities though that I'd like to run through here. One is levels of abstraction, and the other is testing.

## Abstraction

When a function has 0 arguments, it can only do 3 things:

1. It can call other functions
2. It can access data (object state) from itself and return it. 
3. _Nothing_ - if it doesn't call other functions, or query object state, then why bother calling it? 

A _useful_ niladic function is effectively a wrapper for other code, providing a higher level of abstraction.  For example, if we wanted to get a set of data from a report, in a particular format, we could call functions as below.


```python
extract_report_variables_as_json()

# vs

extract_report_variables_from(a_report, FORMAT_JSON)
```

The niladic (0 argument) version reads nicely, and seems simple to understand on the surface. It also provides a higher level of abstraction than the function where you have to provide the report to extract the data from, and the format to return the data in. This (in my mind) is a double-edged sword. The `extractReportVariablesAsJson()` function completely hides where it gets data from, and how the formatting request is passed in (and potentially what other formatting options there might be). 

```python
def extract_report_variables_as_json(self):
    return extract_report_variables_from(self.report, FORMAT_JSON)
```

It's also unclear if there are side effects from this. To figure this out, you need to delve down into the functions to bottom out what they're all doing. For example, to generate the JSON format extract, it may write the data to disk, and this may cause exceptions due to disk space, or permissions problems. You could receive a seemingly unrelated error to the task you're trying to perform due to side effects happening inside an abstracted function.

In the OO world, the general takeaway is that this is a perfect situation. The object provides abstract functions, and they hide the complextity of what is happening under the covers. The main issue I have with niladic functions is the tying of the function to particular state in the object, eg, in our example above the report used will always be `self.report`. To avoid this, the developer has to repeat the abstraction to generalise the functions, all the way down. The developer also has to consider what abstract, and what more detailed (generalised) functions their object might like to expose, and set appropriate access controls on these functions. For this to happen in reality requires a diligence and dedication on the side of the developer to their craft. They must continually strive for clean code - to wrap and abstract out functions. This is a good thing, but not something that's always done by default. 

## Testing

Testing functions has (I believe) an interesting effort curve, depending on the number of arguments in the function, as well some more fundamental constructs of the language you're developing in. 

### Niladic Functions

With niladic functions, you may have to perform more test setup _around_ the function to be able to test it effectively. When a function takes no arguments, you have to setup the data elsewhere for it to operate on. If your function operates on the data it's passed, then you only need to deal with that function to test it. I consider (in general) the test creation effort for niladic functions to be greater than for functions that take arguments.

### Functions with 1+ arguments

I believe the easiest function to test is one with one argument. You can pass in the data you want the function to operate on, receive the result, and check it. This is also true for functions with more than one argument, but the problem with these is that the permutations of possible arguments explode. If you're looking to exhaustively test out a function, then you need to multiply the possible values for each argument together. Lets say you have a function that takes a single boolean. This has 2 possible values you can pass in to it, so it is relatively simple to test exhaustively. If you have a function that takes 3 boolean arguments, then you have 8 possible permutations of the arguments (2 x 2 x 2). Things get crazy when we start looking an other data types such as String or Int. _How many different values can you have for a String?_

### Typed Arguments

One other issue for consideration is if the language you're using supports typed arguments. If it can, a function can define (and ideally enforce) the type of data it can accept. In languages that don't support this, your function could receive data of a different type to what it was expecting.

```python
format = "My Custom Format"
report_data = True
extract_report_variables_from(report_data, format)
```

To generate tests to exhaustively test this is virtually impossible. You won't practically be able to create all the different permutations of options available.

### How to solve?

I think there is a way to get a level of confidence in your functions that receive arguments via testing. Unit tests can provide expected, common scenarios that you want to test for to make sure that the function is behaving under normal scenarios. This should always be done.

If your function can support it (and your language has appropriate libraries), [Property Based Testing](http://blog.jessitron.com/2013/04/property-based-testing-what-is-it.html) can provide a level of confidence that your function can handle all other scenarios. I'm not going to go into it in depth here, but in general, you construct test specifications that will randomly generate test data and pass it to your function. They will repeat this over and over again a number of times, and determine if your function operates as expected in a wider number of scenarios that is humanly possible to code manually. This should give confidence that your function works correctly, even in the face of misuse. 

