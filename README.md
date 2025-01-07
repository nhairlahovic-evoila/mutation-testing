# Mutation Testing in Java

## Introduction

As developers, we understand that testing is a fundamental practice that ensures software behaves as expected, meets requirements, and provides confidence to make changes without breaking functionality. This confidence is one of the most critical aspects—it reassures us during code integration and ensures that everything works as intended before the product reaches the customer.

And how do we achieve this confidence? How can we verify that our tests are good? Since test code is still code, we are just as prone to making mistakes in our test code as we are in our regular code. Someone might have accidentally commented out the asserts in a test or mistakenly copy-pasted the wrong test assertions (Of course, nobody does copy-paste in real-world coding 😉). Or you simply forgot about an edge case.

This is where **mutation testing** comes in. Mutation testing stands out as one of the most effective automated ways to identify test code issues and ensure our tests are effectively checking that the code behaves as expected and meets its intended purpose.

## Why not only Code Coverage?

Usually, one commonly accepted metric for testing is code coverage. It is widely used in pipelines to determine if our code is being tested. However, this metric merely provides a numeric value—indicating how many lines of code are executed during testing—and says nothing about the quality of the tests themselves.

It is even possible to achieve 100% code coverage but still have poor tests or even tests without a single meaningful assertion. This is known as Assertion Free Testing. Mutation testing offers a robust solution by evaluating both the execution of code and the quality of assertions, ensuring a more reliable test suite.

While code coverage has its limitations, it remains a valuable metric in the software testing process. It provides a clear signal about which parts of the codebase are being executed during tests, helping teams identify untested or poorly tested areas. Although it doesn’t measure test quality, it serves as a starting point to understand the testing landscape and prioritize further improvements.

## How is mutation testing working?

Mutation testing works by introducing small changes, or "mutations," into the source code to simulate potential programming errors. These changes can include modifying operators, altering conditions, or tweaking values. The modified code, known as a mutant, is then run against the existing test suite to evaluate whether the tests can detect and fail due to the mutation.

**Killed mutants** occur when a test fails due to a mutation, demonstrating that the test suite successfully caught the issue. **Surviving mutants**, on the other hand, indicate that the test suite did not detect the mutation, highlighting a gap in test coverage for that specific scenario. Developers review surviving mutants to identify weaknesses in the test suite and write additional tests to address them.

Let’s see an example to clarify how mutation testing works in practice.

## Mutation testing in Java with PITest

PITest is a powerful library for performing mutation testing in Java projects. It integrates seamlessly with Maven, allowing developers to incorporate mutation testing into their build pipelines.

### Configuring PITest

To use PITest, include the following dependency in your `pom.xml` file:
```xml
<dependency>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.16.1</version>
</dependency>
```
Next, configure the PITest Maven plugin in your `pom.xml` file:

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.16.1</version>
    <dependencies>
        <dependency>
            <groupId>org.pitest</groupId>
            <artifactId>pitest-junit5-plugin</artifactId>
            <version>1.2.1</version>
        </dependency>
    </dependencies>
    <configuration>
        <targetClasses>
            <param>com.evoila.testing.mutation.*</param>
        </targetClasses>
        <targetTests>
            <param>com.evoila.testing.mutation.*</param>
        </targetTests>
    </configuration>
</plugin>
```

`pitest-junit5-plugin` ensures compatibility with JUnit 5 test suites, which is essential if your project uses JUnit 5 for testing purposes. `targetClasses` specifies the classes in your project that PITest should mutate. 
`targetTests` indicates which tests should be run against the mutated code. In our example, it matches all classes and tests in the root project folder, but this can also be adjusted to focus on specific tests or packages.

### Example Code and Test Suite

Here is a sample implementation of a basic calculator class that provides a simple addition function:
```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}
```

And here are simple JUnit tests to ensure that our implementation works as expected. However, note that one of the assertions is intentionally written incorrectly to mimic an error:
```java
@Test
public void testAddition() {
  Calculator calculator = new Calculator();
  int result = calculator.add(2, 3);
  // Intentional error: this assertion always passes but doesn't validate the method's result
  assertEquals(5, 5);
}
```

With this setup, the tests will run and achieve 100% code coverage. However, the incorrect assertion highlights how a poorly written test can pass without truly validating the functionality of the code.

### Running PITest

To run mutation testing with PITest, execute the following Maven command:
```sh
mvn org.pitest:pitest-maven:mutationCoverage
```

This will generate a detailed report in the target/pit-reports directory, showing killed mutants and surviving mutants. We can check the `com.baeldung.testing.mutation/Calculator.java.html` report for more details about the mutants created:


<img width="885" alt="Screenshot 2025-01-05 at 21 09 12" src="https://github.com/user-attachments/assets/02269eb2-bbba-4304-ba88-1584b37e5570" />


This report shows two mutations in the add method of the Calculator class that survived testing. These include replacing addition with subtraction and replacing the return value with 0. For more details about the PITest mutators, you can check the official [documentation page](https://pitest.org/quickstart/mutators/) link.

The surviving mutants indicate that the test suite failed to detect these changes, highlighting the importance of meaningful assertions to validate the actual behavior of the code.
To achieve 100% code coverage and mutation coverage in this case, you need to correct the test assertion to dynamically validate the method’s output. For example, replace the current assertion assertEquals(5, 5) with assertEquals(5, result).

```java
@Test
public void testAddition() {
    Calculator calculator = new Calculator();
    int result = calculator.add(2, 3);
    assertEquals(5, result);
}
```

### PITest tests configuration

The list of mutators used during mutation testing can be configured in the `pom.xml`. This is a good practice as it helps optimize computing resources by focusing on the most relevant mutations for your project. For example:
```xml
<configuration>
  <targetClasses>
    <param>com.evoila.testing.mutation.*</param>
  </targetClasses>
  <targetTests>
    <param>com.evoila.testing.mutation.*</param>
  </targetTests>
  <mutators>
    <mutator>CONSTRUCTOR_CALLS</mutator>
    <mutator>VOID_METHOD_CALLS</mutator>
    <mutator>RETURN_VALS</mutator>
    <mutator>NON_VOID_METHOD_CALLS</mutator>
  </mutators>
</configuration>
```

PITest provides options to customize testing, such as limiting mutants per class with `maxMutationsPerClass`. For more details, refer to the official [Maven quickstart guide](https://pitest.org/quickstart/maven/).


## Using Mutation Testing in Build Pipelines

Integrating mutation testing into build pipelines can enhance test quality by ensuring that changes in the codebase are adequately validated. Tools like PITest can be configured to run as part of Continuous Integration (CI) processes, automatically generating mutation reports after each build.

Mutation testing is naturally resource and time-intensive. It requires executing a large number of tests for each mutation introduced, which can significantly increase the overall build time. As a result, it’s essential to strategically decide where and when to use mutation testing, focusing on critical areas of the codebase to maximize its benefits.

The author of PITest recommends a more targeted approach to mutation testing. Instead of running it against the entire codebase in a CI job—which can become resource-intensive as the codebase grows—run it frequently on the code you are actively working on during development. This can be done locally or integrated into pull requests using tools like [arcmutate](https://www.arcmutate.com/). For a detailed guide, refer to the author's blog post: [Don't Let Your Code Dry](https://blog.pitest.org/dont-let-your-code-dry/).

## Conclusion

Mutation testing is a great tool for detecting defects in code, ensuring that tests validate actual behavior effectively.
It is not a replacement for code coverage but a complementary method. Use it wisely, as it can be resource-intensive, and focus on critical areas to maximize its benefits.










