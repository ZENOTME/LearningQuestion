## Reference ##
- **Why we need string slice?** <br>
    A string slice is an immutable reference to the data inside the String. 
    It's useful since it ensures the string can't be changed when it exists. In addition, the liter string can be considered as a string slice.
    <br>[More information](https://doc.rust-lang.org/book/ch04-03-slices.html]).
- **What is the Lifetime** <br>
    Every variable including reference has a lifetime. 
    And rust has a *borrow checker* that can compare the lifetime between the reference and variable to ensure ***the lifetime of the reference inside its variable's***.
    But if a function has multiple choice of return, the checker can't decide which reference will return so it can't compare the return reference with its variable. 
    Hence we should provide the lifetime parameter explicitly. From my perspective, it's like a rule or constraint to tell the checker which situation should pass the examination and which not.
    <br>[More information](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)
