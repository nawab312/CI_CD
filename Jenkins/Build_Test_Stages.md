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
