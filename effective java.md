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

