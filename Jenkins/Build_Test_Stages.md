**Unit Tests**

Unit testing focuses on testing small, isolated pieces of code (functions, classes, or modules) to verify their correctness. It ensures that individual components work as expected.

JUnit (Java)
- JUnit is a widely used unit testing framework for Java applications.
- Uses annotations like `@Test`, `@BeforeEach`, and `@AfterEach` to define test cases.
- Supports assertions (`assertEquals`, `assertTrue`, etc.) to validate expected results.
- Can be integrated with build tools like Maven (`mvn test`) and Gradle.

Example JUnit Test
```java
import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;

public class CalculatorTest {
    @Test
    void testAddition() {
        Calculator calc = new Calculator();
        assertEquals(5, calc.add(2, 3));
    }
}
```

PyTest (Python)
- PyTest is a powerful Python unit testing framework.
- Simple function-based testing without needing classes.
- Supports fixtures (`@pytest.fixture`) and parameterized tests.
- Can be run using `pytest test_file.py`.

```bash
import pytest
from calculator import add

def test_addition():
    assert add(2, 3) == 5
```

**Integration Tests**

Integration testing focuses on testing the interaction between multiple components or services. It ensures that different modules work together as expected.
- API Testing: Verifies communication between services using tools like Postman, REST Assured, or PyTest.
- Database Integration Tests: Ensures database operations work correctly.
- Mocking Dependencies: Uses libraries like Mockito (Java) or unittest.mock (Python) to simulate external services.
- Example API Integration Test (PyTest + Requests)
```python
import requests

def test_api_response():
    response = requests.get("https://api.example.com/data")
    assert response.status_code == 200
    assert "data" in response.json()
```

*flaky test*

A flaky test refers to a test that intermittently fails or passes without any changes to the underlying code or environment. These tests are unreliable and cause instability in the CI/CD pipeline, making it difficult to trust the test results. Common Causes of Flaky Tests in CI/CD:
- Network Instability: Network latency, timeouts, or connection issues when tests interact with external services.
- Database Inconsistencies: Test failures caused by inconsistent data states in shared databases, or the inability to reset or control the database state during testing.
- Race Conditions: In parallel or multi-threaded tests, the order of execution might affect the results, causing tests to fail when there is a timing issue.
- Unreliable External Services: Dependencies on third-party services or APIs that may not always be available or responsive.

Fixes for flaky tests in CI/CD:
- Mock External Dependencies: Use mocking frameworks (e.g., WireMock, Mockito) to simulate external APIs and services. This prevents network or service unavailability from affecting test results.
- Implement Retry Mechanism: Automatically retry flaky tests a few times before marking them as failed. This is useful for intermittent network or service issues.
- Increase Timeout Settings: Extend timeout periods for tests that may take longer due to network latency or slow service responses.
- Ensure Test Isolation: Reset the test environment before or after each test (e.g., clear databases, reset caches) to avoid shared state issues between tests.
- Introduce Delays or Synchronization: Add delays or synchronization in multi-threaded tests to avoid race conditions and ensure proper execution order.
- Use In-Memory Databases for Testing: Use fast, isolated, and predictable in-memory databases (like H2 or SQLite) to avoid database state inconsistencies.

**Code Quality & Security Scanning**

Code quality and security scanning help detect issues like code smells(Pieces of code that might not be wrong but could be improved for better maintainability and readability.), security vulnerabilities, and maintainability problems.
- Static Code Analysis: Scans source code for potential bugs and security risks.
- Code Style Enforcement: Ensures adherence to coding standards (e.g., PEP8 for Python, Checkstyle for Java).
- Security Scanning: Identifies vulnerabilities like SQL injection, XSS, and hardcoded credentials.
- Common Tools: ESLint (JavaScript), Checkstyle (Java), Flake8 (Python)

**SonarQube & SAST Tools Integration**

SonarQube and Static Application Security Testing (SAST) tools help automate security and quality checks.

SonarQube
- SonarQube is an open-source tool for static code analysis, detecting: Code smells, Bugs, Security vulnerabilities (OWASP Top 10), Coverage analysis from unit tests

**SAST Tools**

SAST tools analyze source code for security vulnerabilities before execution. Examples:
- SonarQube Security Scanner
- Snyk (Dependency vulnerability scanning)
- Checkmarx
