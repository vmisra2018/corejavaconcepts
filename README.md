##  Functional interface

A _functional interface_ is an interface that specifies exactly one abstract method. .It is annotated by @FunctionalInterface You already know several other functional interfaces in the Java API such as Comparator and Runnable

    **java.util.Comparator**
    @FunctionalInterface
    public interface Comparator<T> {                           
    int compare(T o1, T o2);
    }
    **java.lang.Runnable**
    @FunctionalInterface
    public interface Runnable {                                
    void run();
    }
    **java.awt.event.ActionListener**
    public interface ActionListener extends  EventListener {    
    void actionPerformed(ActionEvent e);
    }
     **java.util.concurrent.Callable**
    @FunctionalInterface
    public interface Callable<V> {
       V call() throws Exception;
    }
    **java.security.PrivilegedAction**
    @FunctionalInterface
    public interface PrivilegedAction<T> {                     
    T run();
    }
    
An interface is still a functional interface if it has many default methods as long as it specifies _only one abstract method_.
**Quiz : Functional interface**

Which of these interfaces are functional interfaces?

    public interface Adder {
    int add(int a, int b);
    }
    public interface SmartAdder extends Adder {
    int add(double a, double b);
    }
    public interface Nothing {
    }
Answer: Adder.  SamrtAdder is not because it also have over

Lambda expressions let you provide the implementation of the abstract method of a functional interface directly inline and _treat the whole expression as an instance of a functional interface_ (more technically speaking, an instance of a _concrete implementation_ of the functional interface).

    Runnable r1 = () ->  System.out.println("Hello World 1");
     Runnable r2 = new Runnable() {                                   
    public void run() {
        System.out.println("Hello World 2");
    }};
    public static void process(Runnable r) {
    r.run();
    }
    process(r1);
    process(r2); 
    process(() ->System.out.println("Hello World 3")

We see above ,process expects Runnable functional interface but we are able to pass lambda implementation of that interface . 
Note that there’s a special rule for a void method invocation defined in the Java Language Specification. You don’t have to enclose a single void method invocation in braces.    

    process(() -> System.out.println("This is awesome"));
### Why can we pass a lambda only where a functional interface is expected?
Most Java programmers are already familiar with the idea of an interface with a single abstract method (for example, for event handling). However, the most important reason is that functional interfaces were already extensively used before Java 8. This means that they provide a nice migration path for using lambda expressions. In fact, if you’ve been using functional interfaces such as Comparator and Runnable or even your own interfaces that happen to define only a single abstract method, you can now use lambda expressions without changing your APIs.

**Quiz 3.3: Where can you use lambdas?**

Which of the following are valid uses of lambda expressions?

        1.  execute(() -> {});
            public void execute(Runnable r) {
              r.run();
            }
            
        2.  public Callable<String> fetch() {
              return () -> "Tricky example ;-)";
            }
            
        3. public interface Predicate<T> {
        boolean test (T t);
       }  
         Predicate<Apple> p = (Apple a) -> a.getWeight();
        

**Answer:**

Only 1 and 2 are valid. 3 is wrong lambda implementation   as interface  `Predicate` has  test method with boolean return type and not Integer:

    @FunctionalInterface
    public interface Predicate<T> {
         boolean test(T t);
      }

  Lambda expressions let you provide the implementation of the abstract method of a functional interface directly inline and  _treat the whole expression as an instance of a functional interface_.
  Java 8 comes with a list of common functional interfaces in the java.util .function package, which includes 

      Predicate < T >, 
      Function<T, R>, 
      Supplier< T >, 
      Consumer< T >, 
      and BinaryOperator<T>

###  Example : How to Implement a Lambda

  Given a `processFile` method  below :
  What should the new processFile method implementation  as lambda look like if you want to read two lines at once? 

    public String processFile() throws IOException {    
      try (BufferedReader br =new BufferedReader(new  FileReader("data.txt"))) { 
                          return br.readLine();<--This line is important.                                                  }
     }

   

#### Step 1: Extract the logic : behavior parameterization
Input is BufferedReader and it should return 2 lines and concatenate as string This current code is limited. Ideally, you’d like to reuse the code doing setup and cleanup and tell the processFile method to perform different actions on the file. Does this sound familiar? Yes, you need to parameterize the behavior of processFile


    String result    = processFile((BufferedReader br) -> br.readLine() + br.readLine());

#### Step 2: Use a functional interface to pass behaviors
Define an abstract method "processRead" to extract behavior

    @FunctionalInterface
    public interface BufferedReaderProcessor {
        String processRead(BufferedReader b) throws IOException;
    }

You can now use this interface as the argument to your new  processFile  method:

    public String processFile(BufferedReaderProcessor p) throws IOException {   ...
    }

#### Step 3: Execute a behavior!
Now the code to return 2 lines is abstracted in process method 

    static class  BufferedReaderProcessorImpl implements BufferedReaderProcessor{
      String processRead(BufferedReader br) throws IOException{
           return br.readLine() + br.readLine();
      }
  }

   
Now pass this functional interface implementation as argument  in the original process method  and then call **processRead** method:


  
<pre>
  public String processFile(<b>BufferedReaderProcessor p</b>) throws IOException {
            try (BufferedReader br =
                           new BufferedReader(new FileReader("data.txt"))) {
                return<b> p.processRead(br)</b>;                                       
            }
        }
</pre>
#### Step 4: Pass processRead as lambdas inside processFile
You can now reuse the processFile method and process files in different ways by passing different lambdas. So only part to be written inside processFile as Lambda while calling  is :
**what is input to functional interface  BufferedReaderProcessor's processRead** method and 
**what is output of BufferedReaderProcessor's processRead**. 
Rest of the stuff is ignored:

    String oneLine =
        processFile((BufferedReader br) -> br.readLine());
    
    String twoLines =
        processFile((BufferedReader br) -> br.readLine() + br.readLine());

## USING FUNCTIONAL INTERFACES
The signature of the abstract method of a functional interface is called a _function descriptor_. 
The signature of the abstract method of a functional interface is called a _function descriptor_. In order to use different lambda expressions, you need a set of functional interfaces that can describe common function descriptors. Several functional interfaces are already available in the Java API such as Comparable, Runnable, and Callable,
 Java 8 has introducing several new functional interfaces inside the java.util.function package. We’ll describe the interfaces Predicate, Consumer, and Function next. A
### Predicate Functional Interface
The java.util.function.Predicate<T> interface defines an abstract method named test that accepts an object of generic type T and returns a boolean. 
  

      @FunctionalInterface
            public interface Predicate<T> {
                boolean test(T t);
            }


####  Requirement:
Let us say we have a filter method which we wrote in some class. 
 <pre>
   public <T> List<T> filter(List<T> list) {
            List<T> results = new ArrayList<>();
            for(T t: list) {
                <b> 
                // write logic to filter/check/validate each item of list
                //it  should  return boolean and it it is true, you add to results</b>
            
                    results.add(t);
                }      
            }
            return results;
        } 
</pre>
We can use Functional interface `Predicate` which has `test` method. It takes T object as input and returns a boolean. Basically we can use it as a check in our generic filter method. So when you call filter as lambda ,we need to reimplement i filter method in our class like below 
####  Reimplementation using Predicate's test method
   <pre>
   public <T> List<T> filter(List<T> list, <b>Predicate<T> p</b>) {
            List<T> results = new ArrayList<>();
            for(T t: list) {
                <b> //Now  here we have called Predicate's
                // test method on each item
                //which will abstract logic to filter/check/validate </b>
                if(<b>p.test(t)</b>{
                    results.add(t);
                }      
            }
            return results;
        } </pre>
So now when we call filter as lambda , it will will have 2 inputs as params:
One input will be `List <T>`  and other will be `<T>`  for Predicate's test and its body :
#### Lambda call  
   <pre> List<Orange> redHeavyOranges = filter(
     <b> //List < T ></b>
       inventoryOranges, 
       <b>//Predicate's test  input and its body with logic </b>
       (Orange o) -> o.getColor() == Color.RED && o.getWeight() > 100 
       );
 </pre>
Some more examples : let is say we have a integer list and we want to filter even numbers 
<pre>List<Integer> numbers = IntStream.range(1, 10).boxed().collect(Collectors.toList());</pre>
    
  <pre>List<Integer> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);</pre>
 
### Consumer Functional Interface
The java.util.function.Consumer<T> interface defines an abstract method named accept that takes an object of generic type T and returns no result (void). 
<pre>@FunctionalInterface
    public interface Consumer<T> {
        void accept(T t);
    }
</pre>
#### Requirement:
 For example, you have a method `forEach` in a class, which takes a list of Integers and applies an operation on each element of that list.
  <pre>public <T> void forEach(List<T> list) {
        for(T t: list) {
          <b>  //here we want to do some processing on each item of input list 
          //which does not modify result
        }
    }
</pre>

Now as lambda implementation of above Consumer interface, we need to pass Consumer as one of params in `forEach` along with `list` as source input and for any processing on list elements will be abstracted in Comsumer's accept method. So now we will rewrite forEach as :
#### Reimplementation using Consumer's accept method
   <pre>public <T> void forEach(List<T> list, <b>Consumer< T > c</b>) {
            for(T t: list) {
             <b> //Now  here we have called 'Consumers
                // accept method on each item
                //which will abstract logic to do some operation on each item 
                //without modifying them 
                  c.accept(t)</b>;
            }
        }
</pre> 
#### Lambda call
So now this is how we will call newly implemented forEach 
   <pre> forEach(
                 <b> // source input is a list  </b>
                 Arrays.asList(1,2,3,4,5),
                 <b>//Consumer's accept method's input + body of accept with logic</b>
                (Integer i) -> System.out.println(i)         
             );
</pre>

As brevity :

    forEach( Arrays.asList(1,2,3,4,5),(Integer i) -> System.out.println(i) );

### Function Functional Interface
The `java.util.function.Function<T, R>` interface defines an abstract method named `apply` that takes an object of generic type T as input and returns an object of generic type R. You might use this interface when you need to define a lambda that maps information from an input object to an output (for         example, extracting the weight of an apple or mapping a string to its length). 

    @FunctionalInterface
        public interface Function< T,  R > {
            R apply(T t);
        }
  #### Requirement:
   Let us say, we have a method map in a class which takes a List <T >
<pre>
public < T , R > List<R> map(List<T> list) {
        List<R> result = new ArrayList<>();
        for(T t: list) {
        <b> //Here we want to do some processing on each item of list 
        //and modify each item</b>
            result.add(t));
        }
        return result;
    }
</pre>
####  Reimplementation using Functions's apply method
<pre>
public < T, R > List<R> map(List<T> list, <b>Function< T,  R > f</b>) {    
     List< R > result = new ArrayList<>();    
           for(T t: list) { 
           <b> 
           //Now  here we have called Function's apply method on each item
           //which will abstract logic to do some operation on each item 
           // modifying each them with some logic  </b>     
               result.add(<b> f.apply(t)</b> );   
             }    
       return result;
 }
 </pre>
#### Lambda call
So now this is how we will call newly implemented map method having Function's apply method
<pre>
 List<Integer> list = 
 map(
      <b> // source input is a list  </b>
        Arrays.asList("lambdas", "in", "action"),
        <b>//Function's apply method's input + body of apply with logic implemented inline</b>
             (String s) -> s.length()                    
          );
</pre>
### Supplier  Functional Interface
The java.util.function.Supplier<T> interface defines an abstract method named 
get whose Predicate Descriptor is  `T -> ()`. This means that there is no input in the lambda definition and the return output is an object of type “T”. The supplier does the opposite of the consumer i.e. it takes no arguments but it returns some value by calling its `get()`method. Do note, the `get()` method may return different values when it is being called more than once. Let’s have a look at a simple code example where the **Supplier** interface is being used.

    @FunctionalInterface
    public interface Supplier<T> {
         T get();
    }

#### Requirement:
 For example, you have a method `displayProduced` in a class, which produces/creates an object T and prints it.
  <pre>public <T> void displayProduced() {
     T t=  create();//create that object
       //print it
        }
    }
</pre>

Now as lambda implementation of above Supplier interface, we need to pass Supplier as one of params in `displayProduced` .We dont need any other input as this is a producer ..So now we will rewrite displayProduced as :
#### Reimplementation using Suppliers's get method

    public static <T> void  displayProduced(Supplier <T> supp) {
   
    	  System.out.println(supplist.get());
    	
    }

 #### Lambda call
So now this is how we will call newly implemented forEach 
   <pre> displayProduced(()->create());
</pre>



### BiFunction  Functional Interface
`java.util.function.BiFunction` is a functional interface. `BiFunction` accepts two arguments and returns a value. While declaring `BiFunction` we need to tell what type of argument will be passed and what will be return type. We can apply our business logic with those two values and return the result. `BiFunction` has function method as `apply(T t, U u)` which accepts two argument.

      BiFunction<Integer,  Integer,  String> biFunction =  (num1, num2)  ->  "Result:"  +(num1 + num2);
        System.out.println(biFunction.apply(20,25));

### BiPredicate Functional Interface
`java.util.function.BiPredicate` is a functional interface which accepts two argument and returns Boolean value. Apply business logic for the values passed as an argument and return the boolean value. `BiPredicate` functional method is `test(Object, Object).` 

    BiPredicate<Integer,  String> condition =  (i,s)-> i>20  && s.startsWith("R"); 
           System.out.println(condition.test(10,"Ram"));
           System.out.println(condition.test(30,"Shyam"));
           System.out.println(condition.test(30,"Ram"));

 ### Functional Interface Cheatsheet

```plain
╔═══════════════════════════════╦═══════════════════════╦═══════════════════════════════════════════════════════════╗
║ Functional Interface          ║      Predicate<T>     ║ Consumer<T>                                               ║
╠═══════════════════════════════╬═══════════════════════╬═══════════════════════════════════════════════════════════╣
║ Predicate: boolean test(T t); ║      T -> boolean     ║ IntPredicate,LongPredicate,DoublePredicate                ║
╠═══════════════════════════════╬═══════════════════════╬═══════════════════════════════════════════════════════════╣
║ Consumer:void accept(T t);    ║       T -> void       ║ IntConsumer,LongConsumer,DoubleConsumer                   ║
╠═══════════════════════════════╬═══════════════════════╬═══════════════════════════════════════════════════════════╣
║ Function:R apply(T t);        ║ T -> R                ║ IntFunction,IntToDoubleFunction,IntToLongFunction,        ║
║                               ║                       ║ LongFunction,LongToDoubleFunction,LongToIntFunction,      ║
║                               ║                       ║ DoubleFunction,DoubleToIntFunction,DoubleToLongFunction,  ║
║                               ║                       ║ ToIntFunction,ToDoubleFunction,ToLongFunction             ║
╠═══════════════════════════════╬═══════════════════════╬═══════════════════════════════════════════════════════════╣
║ Supplier: T get();            ║ () -> T               ║ BooleanSupplier, IntSupplier,LongSupplier, DoubleSupplier ║
╠═══════════════════════════════╬═══════════════════════╬═══════════════════════════════════════════════════════════╣
║ UnaryOperator<T>              ║ T -> T                ║ IntUnaryOperator,LongUnaryOperator,DoubleUnaryOperator    ║
╠═══════════════════════════════╬═══════════════════════╬═══════════════════════════════════════════════════════════╣
║ BinaryOperator<T>             ║ (T, T) -> T           ║ IntBinaryOperator,LongBinaryOperator,DoubleBinaryOperator ║
╠═══════════════════════════════╬═══════════════════════╬═══════════════════════════════════════════════════════════╣
║ BiPredicate<T,U>              ║ (T, U) -> boolean     ║                                                           ║
╠═══════════════════════════════╬═══════════════════════╬═══════════════════════════════════════════════════════════╣
║ BiConsumer<T,U>               ║ (T, U) -> void        ║ ObjIntConsumer<T>,ObjLongConsumer<T>ObjDoubleConsumer<T>  ║
╠═══════════════════════════════╬═══════════════════════╬═══════════════════════════════════════════════════════════╣
║ BiFunction<T,U,R>             ║ BiFunction(T, U) -> R ║ ToIntBiFunction,ToLongBiFunction,ToDoubleBiFunction       ║
╚═══════════════════════════════╩═══════════════════════╩═══════════════════════════════════════════════════════════╝
```


