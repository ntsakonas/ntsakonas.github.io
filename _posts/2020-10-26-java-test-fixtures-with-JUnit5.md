---
layout: post
author: Nick
date: 2020-10-26T22:28:00Z
---
JUnit5 has brought many new features, but one of the least advertised features in one that allows us to make and
use test fixtures in a way that was not easy before.

<!--more-->
### Background

A very common setup in Java unit testing is adding Mockito on top of JUnit, plus the assertion framework of your choice. The most verbose part of a test is the test setup, and everyone can become very creative around that part.

Coming from the Android world, where JUnit4 ruled (at least up to the moment I moved out of Android), 
writting unit tests resulted almost every time in a really heavy setup phase, setting up mocks and mocking 
the appropriate behaviour. When the mock setup exceeded 3-4 lines, it was usually extracted in a separate method,
and when that was becoming too big, it was extracted in utility classes etc. 

The Android team at my current employer is now mostly using Kotlin and they are using test fixtures, which are instantiated on demand at the point of usage. Test fixtures implement very basic behaviour of the mocked objects.
The backend team, where Python is used, test fixtures are heavily used too. 

The differences between languages, have a huge impact on how the fixtures are introduced in a test, but looking at both worlds, and peeking into one specific feature of JUnit5 made me think whether I could make test fixtures I can use in Java tests, but in a similar way Python is doing it.

### JUnit5 Extensions

Enter JUnit5 extensions. You are encouraged to read the [full documentation](https://junit.org/junit5/docs/current/user-guide/#extensions) as it contains some valuable information which unfortunately does not stand out on the first reading. 

The JUnit5 extension model allows us to register extensions in a declarative way just by annotating a test class 
or a test method with `@ExtendWith()`. At runtime the JUnit framework will attemp to "extend" the annotated part with 
the functionality provided. Yes, that reads weird, and it does not make a lot of sense, and this is where you skip it and miss it, because what it does not stand out in the documentation is that
using this feature we can essentially perform dependency injection during test instantiation!

### ParameterResolver is more that what it looks like

Very late in the JUnit5 documentation, under [Parameter Resolution](https://junit.org/junit5/docs/current/user-guide/#extensions-parameter-resolution) it is simply stated that:

_ParameterResolver defines the Extension API for dynamically resolving parameters at runtime._

A very `uninsteresting` part that I had skipped more than once. (note:as I write this, I noticed with an extra reading
another piece of info that I had not noticed, more on that later).

After a few explorations in the documentation, I was wrapping my head around this example

```java

@ExtendWith(RandomParametersExtension.class)
@Test
void test(@Random int i) {
    // ...
}

```

I was wondering what exactly this means and how it is used... it looked like depenency injection but I had to try 
it first.


### Injecting depedencies in test methods

In order to test what I had in mind, I put together some classes with dependencies between them

```java
import lombok.AllArgsConstructor;

@AllArgsConstructor
public class RecordChainer {

    private final BlockSigner stationSigner;

    @Override
    public String chainRecord(String previousBlock, String currentBlock) {
        StringBuilder recordChain = new StringBuilder();
        recordChain.append(previousBlock).append("=").append(currentBlock);
        return stationSigner.signDataBlock(recordChain.toString());
    }
}

public interface BlockSigner {

    String signDataBlock(String message);
}

```

The `BlockSigner` implementation is going to be supported by a cryptographic signing algorithm. 
`RecordChainer` is accepting 2 string blocks, structures them in a specific way, 
and delegates the signing to the `BlockSigner`.

Now, let's attempt to test `RecordChainer`.

### Testing, take 1

_Note:I have never used junit assertions, I find them very 'thin'... [AssertJ](https://assertj.github.io/doc/) has always been my choice_

This is how I usually would write the test, influenced and guided by the restrictions and drawbacks of Junit4 
which after a long time becomes your _standard approach_.


```java
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.anyString;

class RecordChainerTest {

    @Test
    public void verifyFixtureInjection() {

        stationSignerMock = Mockito.mock(BlockSigner.class);
        Mockito.when(stationSignerMock.signDataBlock(anyString())).thenReturn("signature");

        String recordSignature = chainer.chainRecord("previous_block", "next_block");
        assertThat(recordSignature).isEqualTo("signature");

		// Just verify that the signer was called
        Mockito.verify(stationSignerMock, Mockito.times(1)).signDataBlock(anyString());
        // OR
        // Verify that the signer was called with the correct message structure
        Mockito.verify(stationSignerMock, Mockito.times(1)).signDataBlock(eq("previous_block=next_block"));
    }
    
``` 

This is a perfectly acceptable test, because the setup is not that difficult. But this is not always the case,
if the mock needs more configuration, then it has to happen at that point.

We can do better...

### Testing, take 2

The test may be refactored, to extract the setup part (especially if this is re-usable in other tests).
So, something like this would be the next approach (again, influenced by the old JUnit4 approach)


```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.mockito.Mockito;
import org.mockito.Mock;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.anyString;

class RecordChainerTest {

	@Mock
    BlockSigner stationSignerMock;

    @BeforeEach
    void setup(){
        Mockito.when(stationSignerMock.signDataBlock(anyString())).thenReturn("signature");
        // ... more lines with customising stationSignerMock behaviour

    }

    @Test
    void verifyFixtureInjection() {

        String recordSignature = chainer.chainRecord("previous_block", "next_block");
        assertThat(recordSignature).isEqualTo("signature");

		// Just verify that the signer was called
        Mockito.verify(stationSignerMock, Mockito.times(1)).signDataBlock(anyString());
        // OR
        // Verify that the signer was called with the correct message structure
        Mockito.verify(stationSignerMock, Mockito.times(1)).signDataBlock(eq("previous_block=next_block"));
    }
    
``` 

still, pretty good test. My personal disagreement in this style is that the test initialisation phase is very disjoint from the test, and it can be re-used by other tests. That means that at a given time _T0_, if _N_ tests
are using the same setup, the developers may assume that all tests require and depend on the same mock behaviour.
The setup is giving the impression that all test require that.

This may not always be true, and that may lead to incorrect "test fixing" by fixing the failing assertions instead of 
raising concerns about the failures. 

Although I have never used parallel test running, using shared state can wreck havoc between tests, if the tests further modify the behaviour of the mock for their own purposes.

But JUnit5 makes it a bit better...

### Testing, take 3 - the JUnit5 way

JUnit5 provides the `ParameterResolver` interface that allows to "resolve parameters at run time". 
I was intrigued by that so I worked around that, trying to implement a minimum set of classes that could support
this style of testing


```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mockito;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.anyString;

class RecordChainerRefImplTest {

    @ExtendWith(BlockSignerFixture.class)
    @Test
    void verifyFixtureInjection(BlockSigner stationSigner) {
        assertThat(stationSigner).isNotNull().isInstanceOf(BlockSigner.class);

        String recordSignature = chainer.chainRecord("previous_signature", "current_signature");
        assertThat(recordSignature).isEqualTo("previous_signature=current_signature");
        // Verify that the signer was called with the correct message structure
        Mockito.verify(stationSignerMock, Mockito.times(1)).signDataBlock(eq("previous_signature=current_signature"));
    }
    
``` 

In this test, we rely on the framework to provide us with an instance of a `BlockSigner` which is expected to 
be a mock, so that we can further configure it, _but_ is injected already configured with a specific behaviour.

Let's try a basic approach to implement the extension

```java

import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.ParameterContext;
import org.junit.jupiter.api.extension.ParameterResolutionException;
import org.junit.jupiter.api.extension.ParameterResolver;

import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

public abstract class BaseFixture<T> implements ParameterResolver {

    @Override
    public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext) throws ParameterResolutionException {
        return parameterContext.getParameter().getType() == getTargetType();
    }

    abstract protected Class<T> getTargetType();
}
```

This class will allow to create a subclass that provides instances of `<T>` when required.
the `supportsParameter()` is a very interesting, as it is used at runtime to determine if this resolver 
supports the resolution of an argument of type `<T>` in the test execution context.

That means,that if we had a subclass 
```java
public class BlockSignerFixture extends BaseFixture<BlockSigner> {
	...
}
```	

then in the following test, `supportsParameter()` must return `true`

```java
 	@ExtendWith(BlockSignerFixture.class)
    @Test
    void verifyFixtureInjection(BlockSigner stationSigner) {
    }
```	

but if it was misused like in the following test, `supportsParameter()` must return `false`

```java
 	@ExtendWith(BlockSignerFixture.class)
    @Test
    void verifyFixtureInjection(Random Random) {
    }
```	

the `abstract protected Class<T> getTargetType();` was added exactly for that, so that the subclass can 
indicate the provided type.

Having that in mind, let implement the test fixture for the `BlockSigner`

```java
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.ParameterContext;
import org.junit.jupiter.api.extension.ParameterResolutionException;

import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

public class BlockSignerFixture extends BaseFixture<BlockSigner> {

    @Override
    public BlockSigner resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext) throws ParameterResolutionException {
    	// precofingured mock: just return the argument passed
        BlockSigner mock = mock(BlockSigner.class);
        when(mock.signDataBlock(anyString())).thenAnswer(invocation -> invocation.getArgument(0));
        return mock;
    }

    @Override
    protected Class<BlockSigner> getTargetType() {
        return BlockSigner.class;
    }
}
```

`BlockSignerFixture` is a fixture tht provides preconfigured instances of `BlockSigner` to the test methods.
The test methods can of course further mock the injected instance.By doign so, it will not interfere with other
instances of the fixture.

Running the tests with this fixture in place behaves as expected and the test succeeds!
That is a win! 

But the fact that every single class has to explicitly indicate the provided type is just noise.
That is another pattern, commonly found in Java code, and because I find my self using it often, I wanted to
see if I can improve it.

### Extracting the (generic) injected type at runtime

A side-effect of reading 3rd pary code is that some patterns are repeated and you end up believing that this
is probably the right way to do it. For this step, I took the path of thinking ways that this can be achieved
without reading other's code. 

I was really fixated about the fact that in `class BaseFixture<T> `, the _type_ of `T` is available at compile time, but we need the equivalent of `T.getClass()` at runtime. I was typing and deleting some weird code to try to get
`.getClass()` of `T` with compile time safety. I was not getting at it, until I realised that at runtime, the type of `T` will be known, but is there a way to get that given that I am not going to have an instance of that type around?

Fiddling around with reflective methods and a bit of debugging, gave me the result, which honestly, I did not know it was possible!

Here is the revisited code, starting with the `BlockSignerFixture` class which does not have to implement 
the `Class<BlockSigner> getTargetType()` method anymore. It only has to care about providing a proconfigured
mock of the requested type.

```java
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.ParameterContext;
import org.junit.jupiter.api.extension.ParameterResolutionException;

import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

public class BlockSignerFixture extends BaseFixture<BlockSigner> {

    @Override
    public BlockSigner resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext) throws ParameterResolutionException {
    	// precofingured mock: just return the argument passed
        BlockSigner mock = mock(BlockSigner.class);
        when(mock.signDataBlock(anyString())).thenAnswer(invocation -> invocation.getArgument(0));
        return mock;
    }
}
```

the updated `BaseFixture` class has now its way to determine the type of the parametrised type `T` at
runtime, making this part automagic

```java

import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.ParameterContext;
import org.junit.jupiter.api.extension.ParameterResolutionException;
import org.junit.jupiter.api.extension.ParameterResolver;

import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

public abstract class BaseFixture<T> implements ParameterResolver {

    @Override
    public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext) throws ParameterResolutionException {
        return parameterContext.getParameter().getType() == getTargetType();
    }

    final protected Class<T> getTargetType() {
        ParameterizedType genericSuperclass = (ParameterizedType) getClass().getGenericSuperclass();
        return (Class<T>) genericSuperclass.getActualTypeArguments()[0];
    }
}
```

`BlockSignerFixture`  is now implementing the contract that it is providing a fixture for the `BlockSigner` type.
And the happy part is that it is very close to the simple way Python is providing its fixtures.
(This is mostly a way to show Python developers that Java can do it too almost as easily as Python does.)

### Nothing is free of course

This approach has reall worked fine but we need to be aware of the overhead it can lead to, when we start adding 10s or 100s of fixtures. It may not be obvious, but it is very reasonable that if we have a method is annotated with `m` number of `@ExtendsWith` annotations and has `n` number of parameters, the JUnit5 runtime has to examine up to `m x n` number of extensions until it finds the correct one.

for example, in the following test
```java
    @ExtendWith(BlockSignerFixture.class)
    @ExtendWith(BlockEncryptorFixture.class)
    @Test
    void verifyFixtureInjection(BlockSigner stationSignerMock, BlockEncryptor blockEncryptorMock) {
    }
```

in order for `stationSignerMock` parameter to receive the injection, the extensions (`BlockSignerFixture` and `BlockEncryptorFixture`) must be queried in order, until one of them returns `true` from its `supportsParameter()` method, which in turn the JUnit5 extension model will use to get the fixture instance.
After that, the runtime will repeat the check for `blockEncryptorMock` parameter and so on.
This is something we should keep in mind. It is not that bad, as this is test code but if we do that in a few thousand of tests, it may start becoming noticable.

### And a (validating) surprise

As I mentioned, some important details in the documentation do not stand out and can be easily missed or skipped.
It was only during writing this, when I tried to link to the documentation when I noticed this part:

_If you wish to implement a custom ParameterResolver that resolves parameters based solely on the type of the parameter, you may find it convenient to extend the TypeBasedParameterResolver which serves as a generic adapter for such use cases._

What is the `TypeBasedParameterResolver` ? I quickly had a look into the [source code](https://github.com/junit-team/junit5/blob/r5.7.0/junit-jupiter-api/src/main/java/org/junit/jupiter/api/extension/support/TypeBasedParameterResolver.java) and noticed that this class is doing similar tricks with mine to get the runtime 
type of the parametrised type. The fact alone that it is doing more work than my 2 lines, put me into thoughts, and I realised that I have not checked the behaviour of my fixtures when subclasses of the type are requested. Could it work or will it break?

I will test it and write about it...

### Summary

Getting comfortable with your frameworks and libraries may result in using a small subset of its functionality, or not allowing you to experiment in depth with new or not so advertised features. 
In the case of JUnit5, the extension model gives a huge opportunity to make reusable test fixtures and
reduce the noise of test setup on every test. 

Rating of this task
-------------------
Challenge: 7/10
Satisfaction :9/10
Funtime: 10/10