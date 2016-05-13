# Error-Type Specific Catch Blocks
#### Author: Joe Groseclose (jgrosecl49@gmail.com) (@benderTheCrime)
This document aims to outline a proposal for the catching of specific error types in try/catch/finally blocks.

## The Problem
Today, there is not a robust solution for catching errors by type. One must approach handling errors with an "all-or-nothing" strategy, whereby one assumes that all errors are to be treated equally and weighted the same.

This problem is demonstrated below:
```javascript
try {
    // Some code that ostensibly throws an error
} catch (e) {
    switch (e.name) {
    case 'TypeError':
    case 'ReferenceError':
        // Some code that handles these types of errors appropriately
        break;
    default:
        throw e;
    }
}
```

Alternatively, and probably more demonstratively:
```javascript
try {
    // Some code that ostensibly throws an error
} catch (e) {
    // This is not ideal
    if (e instanceof TypeError || e instanceof ReferenceError) {
        // Some code that handles these types of errors appropriately
    } else {
        throw e;
    }
}
```

As is evident, handling of multiple error types is not easy or convenient.

## The Proposal
Handle errors by type in a declarative fashion (similar to the way that Python would catch exceptions specific to any exception type):
```python
try:
    # Some code that ostensibly throws an error
except TypeError:
    # Some code that handles this type of error appropriately
except NameError:
    # Some code that handles this type of error appropriately
except Exception as e:
    # This last part is not a Python best practice, but does demonstrate the use case
    raise e
```

### Simple Case
In JavaScript this time, the proposal is to have catch blocks declare the type of error they are intended to catch and catch errors in serial:
```javascript
try {
    // Some code that ostensibly throws an error
} catch TypeError (e) {
    // Some code that handles this type of error appropriately
} catch ReferenceError (e) {
    // Some code that handles this type of error appropriately
} catch (e) {
    throw e;
}
```

Where the last statement catches all previously unhandled errors. This demonstrates the backwards compatibility of this proposal. If errors are caught in this fashion, then they do not have to be handled with any specificity at all and the existing try/catch composition remains syntactically and semantically correct (catching all error types by default).

### No Generic Case
In the simple case, I demonstrate how to preserve backward compatability by catching all errors in a syntax similar to catching errors in the current specification. However, this proposal also aims to provide a way to only catch specific error types. Thus, if only specific types of errors are handled and a generic case omitted, an error should be thrown:
```javascript
try {
    throw new ReferenceError();
} catch TypeError (e) {
    // Some code that handles this type of error appropriately
}

// Here, a ReferenceError will be thrown
```

### Multiple Errors
Additionally, if one wants to catch many error types, it should be possible to do so with a single catch block. I believe that the syntactically correct way to compose a catch block of this fashion should be the following:

```javascript
try {
    // Some code that ostensibly throws an error
} catch TypeError,ReferenceError (e) {
    // Some code that handles these types of errors appropriately
} catch (e) {
    throw e;
}
```

Alternatively:
```javascript
try {
    // Some code that ostensibly throws an error
} catch (TypeError,ReferenceError) (e) {
    // Some code that handles these types of errors appropriately
} catch (e) {
    throw e;
}
```

Which does the same thing as the previous example with the exception of handling two error types in the same catch block. Obviously, this functionality is being drafted with the intent that one shouldn't have to check the type of the thrown error, so multiple error-type specific catch handling should only be used when there is a known block of code to be executed which handles explicitly both error types.


##### Syntax
The double-parenthesis in the above is admittedly awkward. Below are some other alternatives I have considered:

```javascript
try {
    // Bitwise or
} catch TypeError|ReferenceError (e) {
    // Parenthesis & bitwise or
} catch (TypeError|ReferenceError) (e) {
    // "as"
} catch TypeError as e {
    // Parenthesis & "as"
} catch (TypeError as e) {}
```

### Custom Error Handling
If one so desires, he or she may create a subclass of an error of any type, creating a new, more specific error:
```javascript
class SpecificError extends Error {}
```

Error-Type specific catch handling should also be able to handle these user-defined error types:
```javascript
try {
    // Some code that ostensibly throws an error
} catch SpecificError (e) {
    // Some code that handles this type of error appropriately
} catch (e) {
    throw e;
}
```

As well, the compound case:
```javascript
try {
    // Some code that ostensibly throws an error
} catch SpecificError,TypeError (e) {
    // Some code that handles these types of errors appropriately
} catch (e) {
    throw e;
}
```

### Finally
Not "in conclusion" of course, but `finally`. Once an error-type specific catch block has been executed, the try/catch/finally block should fall through through the finally, even if there are many many handled error types. This preserves the backward compatibility of the try/catch/finally pattern and is the most expressive way to define error-specific logical handlers for errors while allowing errors of any type to perform some subsequent, error-derived action:
```javascript
try {
    // Some code that ostensibly throws an error
} catch TypeError,ReferenceError (e) {
    // Some code that handles these types of errors appropriately
} catch RangeError (e) {
    // Some code that handles this type of error appropriately
} catch (e) {
    console.log(e);
} finally {
    // Some code to execute regardless of which type of error is thrown
}
```


### Conclusion
Based on the usefulness of this feature (and the likely use cases), the minimality of the syntax of the change, and the nature of its complete backward compatibility, I believe this to be a necessary addition to the specification.  