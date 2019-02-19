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