Functional programming in Java means treating functions as data. You can store them, pass them to methods, and execute them later using lambda expressions and functional interfaces.

For example if we are trying to perform different actions like addition subtraction mutliplication division we can write  this method here we can see the operation is also passed as an argument , instead of writing multiple codes for different operations inside the method we can pass what we need to do as a variable using that operation variable which is of type BiFunction. 

```
public static int calculate(
        int a,
        int b,
        BiFunction<Integer, Integer, Integer> operation) {

    return operation.apply(a, b);
}
```

and we can call the method calculate like this 

```
calculate(10, 20, (x, y) -> x + y); // Addition

calculate(10, 20, (x, y) -> x * y); // Multiplication

calculate(10, 20, (x, y) -> x - y); // Subtraction
```

In java when we are using stream api and do things like   .filter(n->n%2!=0). then also we are doing the same thing. notice here we are passing the method signature what we need to do   to the method filter. we can treat functions as a variable we can store them we can return them. 