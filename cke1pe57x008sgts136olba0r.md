## Unit testing frameworks!!! Do they really help us?

Everyone hates testing(including me) as it's boring as well as you might end up writing more code for testing than the actual code which is under test. I learnt it the hard way by bringing down my team's application for a few hours when I tried to refactor code.

Ok, so why you need a framework like JUnit5 for unit testing?

A usual testing cycle will be like this,

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1597859949160/2-aaaSNQq.png)

```
public class Calculator {
       public int add(int a, int b) {
		return a + b;
	}
	
	public int subtract(int a, int b) {
		return a - b;
	}
	
	public int divide(int a, int b) {
		return a / b;
	}
}
``` 

If we want to test the above manually without any testing framework code will look similar to below,


```
public class CalculatorTest {
    public static void main(String[] args) {
       CalculatorTest test = new CalculatorTest();
       test.testAdd();
       test = new CalculatorTest();
       test.testSubtraction();
       test = new CalculatorTest();
       test.testExceptionInDiv();
    }

    void testAdd() {
        Calculator cal = new Calculator();
        int result = cal.add(1,2);
        if (result == 3) {
            System.out.println("Add test passed");
        } else {
            System.out.println("Add test failed. Expected 3 but was " + result);
        }
    }

    void testSubtraction() {
        int result = cal.subtract(2,1);
        if (result == 1) {
            System.out.println("Subtraction test passed");
        } else {
            System.out.println("Subtraction test failed. Expected 1 but was " + result);
        }
    }

    void testExceptionInDiv() {
        try {
           cal.divide(2,0);
           throw new Exception("Fail");
        } catch (ArithmeticException e) {
            System.out.println("Divsion by 0 test passed");
        } catch (Exception e1) {
            System.out.println("Divsion by 0 test failed");
        }
    }
}
```

For each condition, we need to write a conditional block and also make sure the error message is readable. This way of testing also results in special handling for cases like what if one wants to skip a few tests or skip a test if a system property is missing(@EnabledIfSystemProperty annotation in JUnit5) etc. This is where a testing framework like JUnit5 helps.

So the testing cycle will be reduced to this,

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1597860078444/uLg-RTB1Y.png)

I will end with this, a testing framework helps us in writing better tests.
