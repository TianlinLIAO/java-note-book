| Feature                       | Items       | Release |
| ----------------------------- | ----------- | ------- |
| Lambdas                       | Items 42-44 | Java 8  |
| Streams                       | Items 45-48 | Java 8  |
| Optionals                     | Item 55     | Java 8  |
| Default methods in interfaces | Item 21     | Java 8  |
| try-with-resources            | Item 9      | Java 7  |
| @SafeVarargs                  | Item 32     | Java 7  |
| Modules                       | Item 15     | Java 9  |

### Creating and Destroying Objects

#### Item 1: consider static factory methods instead of constructors

#### Item 2: consider a builder when faced with many constructor parameters

When there are many optional parameters. Consider the case of a class representing the Nutrition Facts Label that appears on packaged foods.

##### Bad: telescoping constructor pattern

Provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters.

```java
public class NutritionFacts {
    private final int servingSize; // required
    private final int servings; // required
    private final int calories; // optional
    private final int fat; // optional
    private final int sodium; // optional
    private final int carbohydrate; // optional
    
    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrates) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrates = carbohydrates;
    }
    
}
```

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```
Disadvantages:
* hard to read: wondering what all those values mean
* easy to cause bugs: for example, accidentally reverses two such parameters

##### Bad: JavaBeans pattern

```java
public class NutritionFacts {
    private int servingSize = -1; // required
    private int servings = -1; // required
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;
    
    public NutritionFacts() {}
    
    // Setters
    public void setServingSize(int servingSize) {
		this.servingSize = servingSize;
	}
    
	public void setServings(int servings) {
		this.servings = servings;
	}
    
	public void setCalories(int calories) {
		this.calories = calories;
	}

	public void setFat(int fat) {
		this.fat = fat;
	}

	public void setSodium(int sodium) {
		this.sodium = sodium;
	}

	public void setCarbohydrate(int carbohydrate) {
		this.carbohydrate = carbohydrate;
	}
}
```

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

Disadvantages:

* Because construction is split across multiple calls, a JavaBean may be in an inconsistent state partway through its construction.

* It precludes the possibility of making a class immutable.

* It requires added effort on the part of the programmer to ensure thread safety.

##### Builder pattern

```java
public class NutritionFacts {
	private final int servingSize; 
    private final int servings; 
    private final int calories; 
    private final int fat; 
    private final int sodium; 
    private final int carbohydrate; 
    
    public static class Builder {
    	// required parameters
    	private final int servingSize;
    	private final int servings;
    	
    	// optional parameters
    	private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;
        
        public Builder(int servingSize, int servings) {
        	this.servingSize = servingSize;
        	this.servings = servings;
        }
        
        public Builder calories(int val) {
        	calories = val;
        	return this;
        }
        
        public Builder fat(int val) {
			fat = val;
			return this;
		}
        
        public Builder sodium(int val) {
        	sodium = val;
        	return this;
        }
        
        public Builder carbohydrate(int val) {
        	carbohydrate = val;
        	return this;
        }
        
        public NutritionFacts build() {
        	return new NutritionFacts(this);
        }
    }
    
    public NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
		sodium = builder.sodium;
		carbohydrate = builder.carbohydrate;
	}
}
```

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240,8)
    .calories(100).sodium(35).carbohydrates(27).build();
```

Advantages:

* The client code is easy to write and easy to read.

Disadvantages:

* May affect performance
* More verbose: should be used only if there are enough parameters to make it worthwhile, say four or more. If you may want to add more parameters in the future, it's often better to start with a builder in the first place.

```java
public abstract class Pizza {
	public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
	final Set<Topping> toppings;
	
	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
		
		public T addTopping(Topping topping) {
			toppings.add(Objects.requireNonNull(topping));
			return self();
		}
		
		abstract Pizza build();
		
		// Subclasses must override this method to return "this"
		protected abstract T self();
	}
	
	Pizza(Builder<?> builder) {
		toppings = builder.toppings.clone();
	}
}
```

```java
public class NyPizza extends Pizza{
	public enum Size {SMALL, MEDIUM, LARGE }
	private final Size size;
	
	public static class Builder extends Pizza.Builder<Builder> {
		private final Size size;
		
		public Builder(Size size) {
			this.size = Objects.requireNonNull(size);
		}

		@Override
		Pizza build() {
			return new NyPizza(this);
		}

		@Override
		protected Builder self() {
			return this;
		}
	}

	private NyPizza(Builder builder) {
		super(builder);
		size = builder.size;
	}
}
```

```java
NyPizza pizza = new NyPizza.Builder(SMALL).addTopping(SAUSAGE).addTopping(ONION)
    .build();
```

#### Item 3: Enforce the singleton property with a private constructor or an enum type

A `singleton` is simply a class that is instantiated exactly once.

Common ways to implement: keep the constructor private and exporting a public static member to provide access to the sole instance.

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```
Advantages of public field approach:

* the API makes it clear that the class is a singleton
* it's simpler


```java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
}
```

Advantages of static factory approach:

* It gives you the flexibility to change your mind about whether the class is a singleton without changing its API.
* You can write a generic singleton factory if your application requires it.
* A method reference can be used as a supplier, for example, `Elvis::instance` is a `Supplier<Elvis>`.

**Usually, the public field approach is preferable.**

To make a singleton class serializable, it is not sufficient merely to add `implements Serializable` to its declaration. To maintain the singleton guarantee, declare all instance fields `transient` and provide a `readResolve` method.

```java
// readResolve method to preserve singleton property
private Object readResolve() {
    return INSTANCE;
}
```

A third way to implement a singleton is to declare a single-element enum:

```java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding() { ... }
}
```

#### Item 4: Enforce noninstantiability with a private constructor

Occasionally you'll want to write a class that is just a grouping of static methods and static fields. For example, `java.lang.Math` ,`java.util.Arrays`,`java.util.Collections`.

Such utility classes were not designed to be instantiated: an instance would be nonsensical.

```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```

The `AssertionError` isn't strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class.

#### Item5: Prefer dependency injection to hardwiring resources

Many classes depend on one or more underlying resources. For example, a spell checker depends on a dictionary.

##### Bad: static utility

```java
// Inappropriate use of static utility - inflexible and untestable
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChecker() {} // Noninstantiable
    
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

##### Bad: singleton

```java
// Inappropriate use of singleton - inflexible and testable
public class SpellChecker {
    private final Lexicon dictionary = ...;
    
    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);
    
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

Neither of these approaches is satisfactory, because they assume that there is only one dictionary worth using.

What is required is the ability to support multiple instances of the class (in our example, `SpellChecker`), each of which uses the resource (in our example, the dictionary) desired by the client. A simple pattern that satisfies this requirement is to pass the resource into the constructor when creating a new instance. This is one form of `dependency injection`: the dictionary is a dependency of the spell checker and is injected into the spell checker when it is created.

##### Dependency injection

```java
// Dependency injection provides flexibility and testability
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

A useful variant of the pattern is to pass a resource factory to the constructor. A factory is an object that can be called repeatedly to create instances of type. Such factories embody the Factory Method pattern. The `Supplier<T>` interface is  perfect for representing factories. For example,

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

Although dependency injection greatly improves flexibility and testability, it can clutter up large projects, which typically contain thousands of dependencies. This clutter can be all eliminated by using a dependency injection framework, such as Dagger, Guice, or Spring.

#### Item 6: Avoid creating unnecessary objects

```java
String s = new String("bikini"); // DON'T DO THIS!
```

The argument to the String constructor ("bikini") is itself a String instance. If this usage occurs in a loop or in a frequently invoked method, millions of String instances can be created needlessly. The improved version is simply the following:

```java
String s = "bikini";
```

You can often avoid creating unnecessary objects by using static factory methods (item 1) in preference to constructors on immutable classes that provide both. For example, the factory method `Boolean.valueOf(String)` is preferable to the constructor `Boolean(String)`, which was deprecated in Java 9. The constructor must create a new object each time it's called, while the factory method is never required to do so and won't in practice. In addition to reusing immutable objects, you can also reuse mutable objects if you know they won't be modified.

```java
// Performance can be greatly improved!
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0, 3})" + "(X[CL]|L?X{0, 3})(I[XV]|V?I{0, 3})$");
}
```

`String.matches` is the easiest way to check if a string matches a regular expression, it's not suitable for repeated use in performance-critical situations. The problem is that it internally creates a `Pattern` instance for the regular expression and uses it only once, after which it becomes eligible for garbage collection. To improve the performance, explicitly compile the regular expression into a `Pattern` instance (which is immutable) as part of class initialization, cache it, and reuse the same instance for every invocation of the `isRomanNumeral` method:

```java
// Reusing expensive object for improved performance
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0, 3})" + "(X[CL]|L?{0, 3})(I[XV]|V?I{0, 3})$");
    
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

```java
// Autoboxing will also create unnecessary objects
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```

The variable `sum` is declared as a `Long` instead of a `long`, which means that the program constructs about 2^31^ unnecessary `Long` instances (roughly one for each time the `long i` is added to the `Long sum` ).

#### Item 7: Eliminate obsolete object references 

```java
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
    
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2*size + 1);
    }
}
```

Memory leaks in garbage-collected languages are insidious. If an object reference is unintentionally retained, not only is that object excluded from garbage collection, but so too are any objects referenced by that object, and so on.

The fix for this sort of problem is simple: null out references once they become obsolete.

The corrected version of the `pop` method above looks like this:

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

Nulling out object references should be the exception rather than the norm. The best way to eliminate an obsolete reference is to let the variable that contained the reference fall out of scope.

Whenever a class manages its own memory, the programmer should be alert for memory leaks.

Another common source of memory leaks is caches. Once you put an object reference into a cache, it's easy to forget that it's there and leave it in the cache long after it becomes irrelevant.

A third common source of memory leaks is listeners and other callbacks. If you implement an API where clients register callbacks but don’t deregister them explicitly, they will accumulate unless you take some action. One way to ensure that callbacks are garbage collected promptly is to store only weak references to them, for instance, by storing them only as keys in a `WeakHashMap`.

#### Item 8: Avoid finalizers and cleaners

As of Java 9, finalizers have been deprecated, but they are still being used by the Java libraries. The Java 9 replacement for finalizers is cleaners. Cleaners are less dangerous than finalizers, but still unpredictable, slow, and generally unnecessary. 

 In C++, `destructors` are the normal way to reclaim the resources associated with an object, a necessary counterpart to constructors.
In Java, the `garbage collector` reclaims the storage associated with an object when it becomes unreachable, requiring no special effort on the part of the programmer. C++ destructors are also used to reclaim other nonmemory resources. In Java, a `try-with-resources` or `try-finally` block is used for this purpose.  

#### Item 9: Prefer try-with-resources to try-finally 

The Java libraries include many resources that must be closed manually by invoking a close method. Examples include `InputStream`,`OutputStream`, and `java.sql.Connection`.  
```java
// try-finally: no longer the best way to close resources
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

```java
// try-finally is ugly when used with more than resource
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

Another deficiency of `try-finally`: in the `firstLineOfFile` method, the call to `readLine` could throw an exception due to a failure in the underlying physical device, and the call to close could then fail for the same reason. Under these circumstances, the second exception completely obliterates the first one. There is no record of the first exception in the exception stack trace, which can greatly complicate debugging in real systems—usually it’s the first exception that you
want to see in order to diagnose the problem. 

Java 7 introduced the `try-with-resources` statement. To be usable with this construct, a resource must implement the `AutoCloseable ` interface, which consists of a single void-returning close method.  

```java
// try-with-resources - the best way to close resources
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

```java
// try-with-resources on multiple resources - short and sweet
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src); OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```



### Method Common to All Objects

#### Item 10: Obey the general contract when overriding `equals`

Don’t override the equals method unless you have to: in many cases, the implementation inherited from Object does exactly what you want. If you do override equals, make sure to compare all of the class’s significant fields and to compare them in a manner that preserves all five provisions of the equals contract.  

Equals contract:

* Reflexivity
* Symmetry
* Transitivity
* Consistency
* Non-nullity: For any non-null reference value x, x.equals(null) must return false. 

#### Item 11: Always override hashCode when you override equals 

You must override `hashCode` in every class that overrides equals. If you fail to do so, your class will violate the general contract for `hashCode`, which will prevent it from functioning properly in collections such as `HashMap` and `HashSet`.

```java
public final class PhoneNumber {
	private final short areaCode, prefix, lineNum;

	public PhoneNumber(int i, int j, int k) {
		this.areaCode = rangeCheck(i, 999, "area code");
		this.prefix = rangeCheck(j, 999, "prefix");
		this.lineNum = rangeCheck(k, 9999, "line num");
	}

	private static short rangeCheck(int val, int max, String arg) {
		if (val < 0 || val > max)
			throw new IllegalArgumentException(arg + ": " + val);
		return (short)val;
	}
	
	public boolean equals(Object o) {
		if (o == this) 
			return true;
		if (!(o instanceof PhoneNumber))
			return false;
		PhoneNumber pn = (PhoneNumber)o;
		return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
	}
	
	public static void main(String[] args) {
		Map<PhoneNumber, String> m = new HashMap<>();
		m.put(new PhoneNumber(707, 867, 5309), "Jenny");
		System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
	}
	
}
```

You must override `hashCode` every time you override `equals`, or your program will not run correctly. Your `hashCode` method must obey the general contract specified in Object and must do a reasonable job assigning unequal hash codes to unequal instances.  

#### Item 12: Always override `toString`

While Object provides an implementation of the `toString` method, the string that it returns is generally not what the user of your class wants to see. It consists of the class name followed by an “at” sign (@) and the unsigned hexadecimal representation of the hash code, for example, PhoneNumber@163b91. 

Override Object’s `toString` implementation in every instantiable class you write, unless a superclass has already done so. It makes classes much more pleasant to use and aids in debugging. The `toString` method should return a concise, useful description of the object, in an aesthetically pleasing format. 

#### Item 13: Override clone judiciously

`Cloneable` determines the behavior of Object’s protected clone implementation: if a class implements `Cloneable`, Object’s clone method returns a field-by-field copy of the object; otherwise it throws `CloneNotSupportedException`.

All classes that implement `Cloneable` should override clone with a public method whose return type is the class itself. This method should first call `super.clone`, then fix any fields that need fixing. Typically, this means copying any mutable objects that comprise the internal “deep structure” of the object and replacing the clone’s references to these objects with references to their copies. While these internal copies can usually be made by calling clone recursively, this is not always the best approach. If the class contains only primitive fields or references to immutable objects, then it is likely the case that no fields need to be fixed. There are exceptions to this rule. For example, a field representing a serial number or other unique ID will need to be fixed even if it is primitive or immutable.  

A better approach to object copying is to provide a copy constructor or copy factory.  

A copy constructor is simply a constructor that takes a single argument whose type is the class containing the constructor, for example, 

```java
// copy constructor
public Yum(Yum yum) { ... };
```

A copy factory is the static factory analogue of a copy constructor:

```java
public static Yum newInstance(Yum yum) { ... }
```

The copy constructor approach and its static factory variant have many advantages over Cloneable/clone: they don’t rely on a risk-prone extralinguistic object creation mechanism; they don’t demand unenforceable adherence to thinly documented conventions; they don’t conflict with the proper use of final fields; they don’t throw unnecessary checked exceptions; and they don’t require casts.
Furthermore, a copy constructor or factory can take an argument whose type is an interface implemented by the class. For example, by convention all general-purpose collection implementations provide a constructor whose argument is of
type Collection or Map. Interface-based copy constructors and factories, more properly known as conversion constructors and conversion factories, allow the client to choose the implementation type of the copy rather than forcing the client to accept the implementation type of the original. For example, suppose you have a `HashSet`, s, and you want to copy it as a `TreeSet`. The clone method can’t offer this functionality, but it’s easy with a conversion constructor:
`new TreeSet<>(s)`. 

As a rule, copy functionality is best provided by constructors or factories. A notable exception to this rule is arrays, which are best copied with the clone method. 

#### Item 14: Consider implementing `Comparable` 

Prior editions of this book recommended that `compareTo` methods compare integral primitive fields using the relational operators < and >, and floating point primitive fields using the static methods `Double.compare` and `Float.compare`. In Java 7, static compare methods were added to all of Java’s boxed primitive classes. Use of the relational operators < and > in `compareTo` methods is verbose and error-prone and no longer recommended. 

### Classes and Interfaces

#### Item 15: Minimize the accessibility of classes and members 

Java has many facilities to aid in information hiding. The access control mechanism [JLS, 6.6] specifies the accessibility of classes, interfaces, and members. The accessibility of an entity is determined by the location of its declaration and by which, if any, of the access modifiers (private, protected, and public) is present on the declaration. Proper use of these
modifiers is essential to information hiding. 

The rule of thumb is simple: make each class or member as inaccessible as possible.  

For top-level (non-nested) classes and interfaces, there are only two possible access levels: package-private and public. 

If a package-private top-level class or interface is used by only one class, consider making the top-level class a private static nested class of the sole class that uses it.

For members (fields, methods, nested classes, and nested interfaces), there are four possible access levels, listed here in order of increasing accessibility:

* private—The member is accessible only from the top-level class where it is
  declared.
* package-private—The member is accessible from any class in the package
  where it is declared. Technically known as default access, this is the access
  level you get if no access modifier is specified (except for interface members,
  which are public by default).
* protected—The member is accessible from subclasses of the class where it is
  declared (subject to a few restrictions [JLS, 6.6.2]) and from any class in the
  package where it is declared.
* public—The member is accessible from anywhere. 

With the exception of public static final fields, which serve as constants, public classes should have no public fields. Ensure that objects referenced by public static final fields are immutable. 

Note that a nonzero-length array is always mutable, so it is wrong for a class to have a public static final array field, or an accessor that returns such a field. If a class has such a field or accessor, clients will be able to modify the contents of the array. This is a frequent source of security holes. Alternatively, you can make the array private and add a public method that returns a copy of a private array.

As of Java 9, there are two additional, implicit access levels introduced as part of the module system. A module is a grouping of packages, like a package is a grouping of classes. A module may explicitly export some of its packages via
export declarations in its module declaration (which is by convention contained in a source file named module-info.java). 

#### Item 16: In public classes, use accessor methods, not public fields 

Public classes should never expose mutable fields. It is less harmful, though still questionable, for public classes to expose immutable fields. It is, however, sometimes desirable for package-private or private nested classes to expose fields, whether mutable or immutable. 

#### Item 17: Minimize mutability

The Java platform libraries contain many immutable classes, including `String`, the boxed primitive classes, and
`BigInteger` and `BigDecimal`. There are many good reasons for this: Immutable classes are easier to design, implement, and use than mutable classes. They are less prone to error and are more secure. 

To make a class immutable, follow these five rules:

* Don’t provide methods that modify the object’s state (known as mutators). 
* Ensure that the class can’t be extended.  Preventing subclassing is generally accomplished by making the class final, but there is an alternative: Instead of making an immutable class final, you can make all of its constructors private or package-private and add public static factories in place of the public constructors.
* Make all fields final.  
* Make all fields private. 
* Ensure exclusive access to any mutable components. 

#### Item 18: Favor composition over inheritance 

To summarize, inheritance is powerful, but it is problematic because it violates encapsulation. It is appropriate only when a genuine subtype relationship exists between the subclass and the superclass. Even then, inheritance may lead to fragility if the subclass is in a different package from the superclass and the superclass is not designed for inheritance. To avoid this fragility, use composition and forwarding instead of inheritance, especially if an appropriate interface to implement a wrapper class exists. Not only are wrapper classes more robust than subclasses, they are also more powerful. 

#### Item 19: Design and document for inheritance or else prohibit it 

#### Item 20: Prefer interfaces to abstract classes 

To summarize, an interface is generally the best way to define a type that permits multiple implementations. If you export a nontrivial interface, you should strongly consider providing a skeletal implementation to go with it. To the extent possible, you should provide the skeletal implementation via default methods on the interface so that all implementors of the interface can make use of it. That said, restrictions on interfaces typically mandate that a skeletal implementation take the form of an abstract class. 

#### Item 21: Design interfaces for posterity 

Even though default methods are now a part of the Java platform, it is still of the utmost importance to design interfaces with great care. While default methods make it possible to add methods to existing interfaces, there is great risk in doing so.  

#### Item 22: Use interfaces only to define types 

```java
// Constant interface antipattern - do not use!
public interface PhysicalConstants {
	static final double AVOGADROS_NUMBER = 6.022_140_857e23;
	
	static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
	
	static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

**The constant interface pattern is a poor use of interfaces. **It represents a commitment: if in a future release the class is modified so that it no longer needs to use the constants, it still must implement the interface to ensure binary compatibility. If a nonfinal class implements a constant interface, all of its subclasses will have their namespaces polluted by the constants in the interface. 

If you want to export constants, there are several reasonable choices. If the constants are strongly **tied to an existing class or interface**, you should add them to the class or interface. For example, all of the boxed numerical primitive
classes, such as `Integer` and `Double`, export `MIN_VALUE` and `MAX_VALUE` constants. If the constants are best viewed as members of an enumerated type, you should **export them with an enum type** (Item 34). Otherwise, you should **export the constants with a noninstantiable utility class** (Item 4). Here is a utility class version of the `PhysicalConstants` example shown earlier: 

```java
public class PhysicalConstants {
    private PhysicalConstants() {} // Prevents instantiation
    
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
    public static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
    public static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

Incidentally, note the use of the **underscore character (_) in the numeric literals**. Underscores, which have been legal since Java 7, have no effect on the values of numeric literals, but can make them much easier to read if used with
discretion.  

In summary, interfaces should be used only to define types. They should not be used merely to export constants. 

#### Item 23: Prefer class hierarchies to tagged classes 

```java
// Tagged class - vastly inferior to a class hierarchy!
class Figure {
    enum Shape { RECTANGLE, CIRCLE};
    
    // Tag field - the shape of this figure
    final Shape shape;
    
    // These fields are used only if shape is RECTANGLE
    double length;
    double width;
    
    // This field is used only if shape is CIRCLE
    double radius;
    
    // Constructor for circle
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }
    
    // Constructor for rectangle
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }
    
    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return MATH.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

Tagged classes are verbose, error-prone, and efficient.

```java
// Class hierarchy replacement for a tagged class
abstract class Figure() {
    abstract double area();
}

class Circle extends Figure {
    final double radius;
    
    Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;
    
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override
    double area() {
        return length * width;
    }
}
```

When you encounter an existing class with a tag field, consider refactoring it into a hierarchy. 

#### Item 24: Favor static member classes over nonstatic 

A nested class is a class defined within another class. **A nested class should exist only to serve its enclosing class**. If a nested class would be useful in some other context, then it should be a top-level class. There are four kinds of nested classes: static member classes, nonstatic member classes, anonymous classes, and local classes.  

* static member class

It is best thought of as an ordinary class that happens to be declared inside another class and has access to all of the enclosing class’s members, even those declared private. A static member class is a static member of its enclosing class and obeys the same accessibility rules as other static members. If it is declared private, it is accessible only within the enclosing class, and so forth. 

One common use of a static member class is as a public helper class, useful only in conjunction with its outer class. For example, consider an enum describing the operations supported by a calculator (Item 34). The `Operation` enum should be a public static member class of the `Calculator` class. Clients of `Calculator` could then refer to operations using names like `Calculator.Operation.PLUS` and `Calculator.Operation.MINUS`.

* nonstatic member class

**Each instance of a nonstatic member class is implicitly associated with an enclosing instance of its containing class. **Within instance methods of a nonstatic member class, you can invoke methods on the enclosing instance or obtain a reference to the enclosing instance using the qualified `this` construct.



