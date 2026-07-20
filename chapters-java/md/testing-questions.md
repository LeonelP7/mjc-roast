# TESTING

## Fundamentals

- What is unit testing, and how does it differ from integration testing?
- What makes a good unit test? (think: independence, speed, repeatability)
- What is the AAA pattern (Arrange-Act-Assert)?
- What is Test-Driven Development (TDD)? Can you describe the red-green-refactor cycle?
- What is code coverage, and why isn't 100% coverage the same as "well tested"?
- What is a test pyramid, and why is it shaped that way?

## JUnit

- What is JUnit, and what version does the Spring ecosystem currently use by default?
- What's the purpose of `@Test`, `@BeforeEach`, `@AfterEach`, `@BeforeAll`, and `@AfterAll`?
- What's the difference between `@BeforeEach`/`@AfterEach` and `@BeforeAll`/`@AfterAll`?
- How do you write a parameterized test in JUnit 5? What's `@ParameterizedTest` for?
- What does `@Disabled` do, and when would you use it?
- What's the difference between `assertEquals`, `assertTrue`, and `assertThrows`?
- How do you test that a method throws a specific exception?
- What is `@DisplayName` used for?

## Mocking

- What is a mock, and why do we use mocks in unit tests?
- What's the difference between a mock, a stub, and a spy?
- What is Mockito, and how do you create a mock with it?
- What's the difference between `@Mock`, `@Spy`, and `@InjectMocks`?
- What does `when(...).thenReturn(...)` do?
- What's the difference between `mock()` and `@Mock` (Mockito annotation vs. manual creation)?
- How do you verify that a mocked method was called, and how many times?
- What is `ArgumentCaptor` used for?
- Why shouldn't you mock the class you're actually testing?

## Assertions & Libraries

- Have you used AssertJ? What advantage does it offer over plain JUnit assertions?
- What's the difference between `assertEquals(expected, actual)` and `assertEquals(actual, expected)` — does order matter?

## Spring Boot Testing

- What does `@SpringBootTest` do, and when would you use it vs. a plain unit test?
- What is `@WebMvcTest`, and how does it differ from `@SpringBootTest`?
- What is `@DataJpaTest` used for?
- What's the purpose of `MockMvc`?
- What's the difference between `@MockBean` and `@Mock`?
- How do you test a REST controller without starting a full embedded server?
- What is `TestRestTemplate` or `WebTestClient` used for?

## Test Types & Strategy

- What is an integration test, and how does it differ from a unit test in scope and setup?
- What is a "test double," and what are the different kinds (dummy, fake, stub, spy, mock)?
- What is regression testing?
- What's the difference between black-box and white-box testing?
- Why is it considered bad practice for unit tests to depend on a real database or external API?
- What is Testcontainers, and what problem does it solve?

## General / Behavioral

- How do you decide what to test and what not to test?
- What makes tests "flaky," and how would you deal with a flaky test?
- Have you worked with any CI pipeline that runs tests automatically? What happens if a test fails?
