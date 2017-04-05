# Code Styling

Styling is very important in maintaining a codebase with multiple (and large amounts) of contributors.
A set of code style guidelines are enforced by this project, most of which are checked by
the static analysis tool checkstyle. You can find the general style guidelines used by this project at:
 
- [Google Java style guide](https://google.github.io/styleguide/javaguide.html)
  The only difference is that this project uses the default 4-8 indentation style instead of the 2-4.
- [Android style guide](http://source.android.com/source/code-style.html)


## General

1. Space for indents
2. 4 spaces per indent
3. K and R styled bracing
4. Camel cased members and vars
5. Capital cased classes and constructs

```java
class MyActivity extends Activity {
    @Override
    public void onCreate() {
        super.onCreate();
        // Do more stuff
    }
}
```

#### Copyright Header

1. Add copyright header on top of files where code is authored
2. Prepend to files such as .java, .xml, .gradle, others

```java
/*
 * Copyright 2017 Vandolf Estrellado
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *        http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/**
 * My activity javadoc
 */
class MyActivity extends Activity {
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
  ~ Copyright 2017 Vandolf Estrellado
  ~
  ~ Licensed under the Apache License, Version 2.0 (the "License");
  ~ you may not use this file except in compliance with the License.
  ~ You may obtain a copy of the License at
  ~
  ~        http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
  -->
```

#### Classes

1. mVar pattern for member variables is not required.
2. In each class their organization of declarations should be as follows in order

    - Member vars, starting with static finals, static, public, private
    - Member var blocks such as onClickListener definitions
    - Static methods
    - @Overrides in groups such as Activity lifecycle
    - Activity @Override methods should be placed in proper lifecycle order
    - Public methods
    - Protected methods
    - Private methods
    - Inner classes
    
3. Prefer to instantiate interfaces as own private final class members.

#### Javadoc

All class level Javadoc is required. It is strongly recommended to document public methods from
interfaces and classes, especially those that can be reused for multiple purposes.


## Testing

A set of standard testing conventions is used throughout this project.

#### Method Naming and Structure

* [Test method naming convention](http://osherove.com/blog/2005/4/3/naming-standards-for-unit-tests.html)
* [Test sections separation convention (given-when-then)](http://www.javacodegeeks.com/2015/01/given-when-then-in-java.html)

The testing convention can be summarized by the below template:

```java
@Test
public void methodName_whenThisIsTheCase_thenDoesThis() throws Exception {
    // GIVEN
    // WHEN
    // THEN
}
```

These conventions are followed to keep the test code more readable/understandable, maintainable, and 
atomic (specific to a scenario per method).

Other things to note:

- The GIVEN WHEN THEN in comments are present to separate the 3 different sections for clarity.
- The "when" and "then" part of the method name is enough to describe the test case in detail. 
- The "given" part is implied by the "when" and "then" and made explicit in the GIVEN block inside the test method.
- The "then" is not used in the test method names.
- There are cases where the "when" part of the method name is omitted because it does not make sense to add it. 
- The given-when-then plugin is not used due to its limitations and unnecessary existence.
- Test method names are not prefixed with "test" to follow DRY principles since JUnit 4 no longer needs it.
- It is recommended to edit your Android Studio generate JUnit4 test method template to include as 
  much of these testing conventions as possible for easier test method generation.

#### Method Ordering

To increase the readability and maintainability of the tests, the test methods are ordered 
in the order they are declared in the actual test subject class.

```java
public class MyClass {
    void method1() {...}
    void method2() {...}
}

public class MyClassTest {
    @Test
    public void method1_whenThisIsTheCase_thenDoesThis() {...}
    
    @Test
    public void method2_whenThisIsTheCase_thenDoesThis() {...}
}
```

#### Field Naming and Ordering

The object under test is declared as a class field named "testSubject" to clearly identify the test's subject.

Mockito's @InjectMock and @Mock annotations are used. The fields of the tests are named the same
as the fields of the test subject being mocked.

```java
public class MyClass {
    private final Object myObject1;
    private final Object myObject2;
}

public class MyClassTest {

    @InjectMocks
    private MyClass testSubject;
    
    @Mock
    private Object myObject1;
    
    @Mock
    private Object myObject2;
    
    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
    }
}
```

#### Testing Private Fields

Reflection is used to get a privately declared and constructed member variable instead of
providing getters or modifying the access modifier of the field to be tested 
(and annotating with @VisibleForTesting). 

```java
public class MyClass {
    private final MyCallback myCallback = new MyCallback() {
        @Override
        public void onSuccess() {
            ...
        }
    };
}

public class MyClassTest {

    @InjectMocks
    private MyClass testSubject;
    
    private MyCallback myCallback;
    
    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
        myCallback = (MyCallback) Whitebox.getInternalState(testSubject, "myCallback");
    }
    
    @Test
    public void myCallback_onSuccess_whenThisIsTheCase_thenDoesThis() {...}
}
```

This is done in order to keep the codebase cleaner and shorter. The caveat is that renaming these fields
may break tests. However, this is a small price to pay for a better world. 

A couple of things to note:

- "Why do we need to test private members?" -> "Saying that private methods do not need testing is 
  like saying a car is fine as long as it drives okay" :)
- This is not a question about access modifiers. Rather, the whole point of using the Whitebox is 
  simply to avoid increasing visibility solely for testing purposes. 

The test method names of private field method tests are prepended with the name of the private field.

```java
@Test
public void myCallback_onSuccess_whenThisIsTheCase_thenDoesThis() {...}
```


## Checkstyle Style Enforcement and Static Analysis

Checkstyle is integrated as part of the static analysis check to ensure that standard Java code 
style rules are followed. It does not, however, contain checks for Android specific code style. 
It is strongly recommended to verify that your code passes the static analysis checks before committing.

To run checkstyle 

```bash
./gradlew checkstyle
``` 

To run all static analysis checks (including checkstyle) for all build variants,

```
./gradlew checkQuality
```

The above will run checkstyle, findbugs, PMD, and lint for all build variants. 
To run build variant specific static analysis checks,

```
./gradlew check<build_variant>Quality
```

For example, to run static analysis checks for the MVP Debug build variant,

```
./gradlew checkMvpDebugQuality
```

You may run individual static analysis checks separately. 
To see the list of all gradle tasks,

```
./gradlew tasks
```