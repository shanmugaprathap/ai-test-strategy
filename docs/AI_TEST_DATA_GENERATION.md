# AI-Powered Test Data Generation

A comprehensive guide to generating test data using AI and machine learning.

## Table of Contents

- [Overview](#overview)
- [Traditional vs AI Data Generation](#traditional-vs-ai-data-generation)
- [LLM-Based Data Generation](#llm-based-data-generation)
- [Synthetic Data for Privacy](#synthetic-data-for-privacy)
- [Edge Case Generation](#edge-case-generation)
- [Tools and Libraries](#tools-and-libraries)
- [Implementation Examples](#implementation-examples)
- [Best Practices](#best-practices)

---

## Overview

AI-powered test data generation offers:
- **Realistic data** that mimics production patterns
- **Edge cases** that humans might miss
- **Privacy compliance** with synthetic data
- **Scalability** for large test suites
- **Context awareness** for domain-specific data

---

## Traditional vs AI Data Generation

| Approach | Pros | Cons |
|----------|------|------|
| **Hardcoded** | Simple, predictable | Limited, high maintenance |
| **Faker Libraries** | Good variety, fast | Rule-based, no context |
| **Production Copies** | Realistic | Privacy risks, stale |
| **AI-Generated** | Context-aware, edge cases | API costs, latency |

### When to Use AI

✅ Use AI for:
- Complex domain-specific data
- Edge case discovery
- Privacy-sensitive scenarios
- Multi-field correlations

❌ Avoid AI for:
- Simple random values
- High-volume data (use Faker)
- Deterministic test data

---

## LLM-Based Data Generation

### Basic Prompting

```python
import openai

def generate_test_users(count: int, context: str) -> list:
    prompt = f"""
    Generate {count} realistic test user profiles for {context}.

    Include:
    - Full name (diverse ethnicities)
    - Email (varied formats)
    - Phone (international formats)
    - Address (include edge cases)
    - Birth date (various ages, edge dates)

    Return as JSON array.
    """

    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.8  # Higher for variety
    )

    return json.loads(response.choices[0].message.content)
```

### Java Implementation

```java
public class AIDataGenerator {

    private static final OpenAI openai = new OpenAI(System.getenv("OPENAI_API_KEY"));

    public static List<User> generateUsers(int count, String context) {
        String prompt = String.format("""
            Generate %d realistic test users for %s.
            Include diverse names, valid emails, international phones.
            Return as JSON array with fields: name, email, phone, address, birthDate.
            """, count, context);

        String response = openai.chat(prompt);
        return parseUsers(response);
    }

    public static String generateEmail(String type) {
        Map<String, String> prompts = Map.of(
            "valid", "Generate a valid, realistic email address",
            "edge", "Generate a valid but unusual email (subdomains, plus addressing, long)",
            "international", "Generate a valid email with international domain",
            "corporate", "Generate a corporate email format (firstname.lastname@company.com)"
        );

        return openai.chat(prompts.get(type)).trim();
    }
}
```

### Structured Output with JSON Schema

```python
from pydantic import BaseModel
from typing import List

class Address(BaseModel):
    street: str
    city: str
    state: str
    zip_code: str
    country: str

class User(BaseModel):
    name: str
    email: str
    phone: str
    address: Address
    birth_date: str

def generate_structured_users(count: int) -> List[User]:
    prompt = f"""
    Generate {count} test users.

    Return JSON matching this schema:
    {{
        "users": [
            {{
                "name": "string",
                "email": "string",
                "phone": "string",
                "address": {{
                    "street": "string",
                    "city": "string",
                    "state": "string",
                    "zip_code": "string",
                    "country": "string"
                }},
                "birth_date": "YYYY-MM-DD"
            }}
        ]
    }}
    """

    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        response_format={"type": "json_object"}
    )

    data = json.loads(response.choices[0].message.content)
    return [User(**u) for u in data["users"]]
```

---

## Synthetic Data for Privacy

### GDPR-Compliant Generation

```java
public class PrivacyCompliantGenerator {

    /**
     * Generate synthetic data that maintains statistical properties
     * without containing real PII
     */
    public static Customer generateSyntheticCustomer() {
        String prompt = """
            Generate a completely fictional customer record.

            Requirements:
            - Name must not match any real person
            - SSN format XXX-XX-XXXX but not a real SSN
            - Credit card must pass Luhn check but not be real
            - Address should use real street names but fake numbers

            Return JSON with: name, ssn, creditCard, address, email, phone
            """;

        return parseCustomer(callAI(prompt));
    }

    /**
     * Generate data matching production distribution without real values
     */
    public static List<Transaction> generateSyntheticTransactions(
            TransactionDistribution distribution, int count) {

        String prompt = String.format("""
            Generate %d synthetic financial transactions matching this distribution:
            - Average amount: $%.2f
            - Std deviation: $%.2f
            - Categories: %s
            - Time range: %s to %s

            Data must be completely fictional but statistically similar.
            Return as JSON array.
            """,
            count,
            distribution.avgAmount(),
            distribution.stdDev(),
            distribution.categories(),
            distribution.startDate(),
            distribution.endDate()
        );

        return parseTransactions(callAI(prompt));
    }
}
```

### Data Masking with AI

```python
def mask_sensitive_data(real_data: dict) -> dict:
    """
    Use AI to generate realistic replacements for sensitive fields
    while maintaining data relationships
    """
    prompt = f"""
    Given this data structure (with sensitive values redacted):
    {json.dumps(redact_values(real_data))}

    Generate realistic fake values that:
    1. Maintain the same format
    2. Preserve relationships (e.g., city matches zip code)
    3. Are completely fictional

    Return the same structure with fake values.
    """

    return json.loads(call_ai(prompt))
```

---

## Edge Case Generation

### Automatic Edge Case Discovery

```java
public class EdgeCaseGenerator {

    /**
     * AI analyzes field requirements and generates edge cases
     */
    public static List<String> generateEdgeCases(String fieldName, String requirements) {
        String prompt = String.format("""
            Field: %s
            Requirements: %s

            Generate edge case test values including:
            1. Boundary values (min, max, min-1, max+1)
            2. Special characters
            3. Unicode/international characters
            4. Empty/null/whitespace
            5. Very long values
            6. Format variations
            7. Injection attempts (SQL, XSS, command)

            Return as JSON array of objects: {"value": "...", "category": "...", "shouldPass": true/false}
            """, fieldName, requirements);

        return parseEdgeCases(callAI(prompt));
    }

    /**
     * Generate edge cases for email field
     */
    public static List<TestCase> emailEdgeCases() {
        return generateEdgeCases("email", """
            - Must be valid email format
            - Max 254 characters
            - Domain must exist (not validated)
            """);
    }

    // Example output:
    // {"value": "a@b.co", "category": "minimum_valid", "shouldPass": true}
    // {"value": "user+tag@sub.domain.co.uk", "category": "complex_valid", "shouldPass": true}
    // {"value": "user@localhost", "category": "no_tld", "shouldPass": false}
    // {"value": "user@.com", "category": "invalid_domain", "shouldPass": false}
    // {"value": "<script>@evil.com", "category": "xss_attempt", "shouldPass": false}
    // {"value": "a".repeat(250) + "@test.com", "category": "too_long", "shouldPass": false}
}
```

### Domain-Specific Edge Cases

```python
def generate_domain_edge_cases(domain: str, entity: str) -> list:
    """
    Generate edge cases specific to business domain
    """
    prompt = f"""
    Domain: {domain}
    Entity: {entity}

    Generate edge cases that a domain expert would think of.

    For example, if domain is "banking" and entity is "transaction":
    - Transaction at midnight (date boundary)
    - Transaction on bank holiday
    - Transaction in different timezone
    - Micro-transaction (0.01)
    - Transaction at daily limit
    - Duplicate transaction detection

    Generate 15 domain-specific edge cases.
    Return as JSON with: scenario, test_data, expected_behavior
    """

    return json.loads(call_ai(prompt))

# Example usage
edge_cases = generate_domain_edge_cases(
    domain="e-commerce",
    entity="shopping_cart"
)
# Returns cases like:
# - Cart with 1000 items (performance)
# - Item goes out of stock during checkout
# - Price changes during session
# - Coupon expires mid-checkout
# - Currency conversion edge cases
```

---

## Tools and Libraries

### Comparison Matrix

| Tool | Type | Best For | Pricing |
|------|------|----------|---------|
| **OpenAI GPT-4** | LLM API | Complex, context-aware data | Pay per token |
| **Anthropic Claude** | LLM API | Long context, structured data | Pay per token |
| **Faker** | Library | High-volume simple data | Free |
| **Mimesis** | Library | Localized fake data | Free |
| **SDV** | ML Library | Synthetic tabular data | Free |
| **Gretel.ai** | Platform | Privacy-safe synthetic data | Freemium |
| **Mostly AI** | Platform | Enterprise synthetic data | Enterprise |
| **Tonic.ai** | Platform | Database subsetting + synthesis | Enterprise |

### Faker + AI Hybrid Approach

```java
public class HybridDataGenerator {

    private final Faker faker = new Faker();
    private final AIClient ai = new AIClient();

    /**
     * Use Faker for volume, AI for intelligence
     */
    public List<User> generateUsers(int count) {
        // Generate base data with Faker (fast, free)
        List<User> users = IntStream.range(0, count)
            .mapToObj(i -> User.builder()
                .name(faker.name().fullName())
                .email(faker.internet().emailAddress())
                .phone(faker.phoneNumber().phoneNumber())
                .build())
            .collect(Collectors.toList());

        // Enhance 10% with AI edge cases
        int edgeCaseCount = count / 10;
        List<User> edgeCases = ai.generateEdgeCaseUsers(edgeCaseCount);

        // Replace random entries with edge cases
        Random random = new Random();
        for (User edgeCase : edgeCases) {
            int index = random.nextInt(users.size());
            users.set(index, edgeCase);
        }

        return users;
    }
}
```

---

## Implementation Examples

### Complete Test Data Service

```java
@Service
public class TestDataService {

    private final OpenAIClient openai;
    private final Faker faker;
    private final Cache<String, List<?>> cache;

    /**
     * Smart data generation based on context
     */
    public <T> List<T> generate(Class<T> type, int count, DataContext context) {
        // Check cache first
        String cacheKey = type.getName() + "_" + context.hashCode();
        if (cache.containsKey(cacheKey)) {
            return (List<T>) cache.get(cacheKey);
        }

        List<T> data;

        if (context.requiresAI()) {
            // Use AI for complex/edge cases
            data = generateWithAI(type, count, context);
        } else {
            // Use Faker for simple cases
            data = generateWithFaker(type, count);
        }

        cache.put(cacheKey, data);
        return data;
    }

    /**
     * Generate correlated data sets
     */
    public TestDataSet generateCorrelatedData(String scenario) {
        String prompt = String.format("""
            Generate a complete test data set for scenario: %s

            Include correlated entities:
            - Users (who make orders)
            - Products (that are ordered)
            - Orders (linking users and products)
            - Payments (for orders)

            Ensure referential integrity and realistic relationships.
            Return as JSON with separate arrays for each entity.
            """, scenario);

        return parseDataSet(openai.chat(prompt));
    }
}
```

### DataProvider Integration

```java
public class AIDataProvider {

    @DataProvider(name = "aiGeneratedUsers")
    public Object[][] generateUserTestData() {
        List<Map<String, Object>> users = AIDataGenerator.generateUsers(10, "e-commerce checkout");

        return users.stream()
            .map(user -> new Object[]{user})
            .toArray(Object[][]::new);
    }

    @DataProvider(name = "edgeCaseEmails")
    public Object[][] generateEmailEdgeCases() {
        List<EdgeCase> cases = AIDataGenerator.generateEdgeCases("email", "standard email validation");

        return cases.stream()
            .map(c -> new Object[]{c.getValue(), c.shouldPass()})
            .toArray(Object[][]::new);
    }
}

// Usage in test
public class RegistrationTest {

    @Test(dataProvider = "edgeCaseEmails", dataProviderClass = AIDataProvider.class)
    public void testEmailValidation(String email, boolean shouldPass) {
        RegistrationPage page = new RegistrationPage();
        page.enterEmail(email);
        page.submit();

        if (shouldPass) {
            assertThat(page.getErrors()).isEmpty();
        } else {
            assertThat(page.getErrors()).contains("Invalid email");
        }
    }
}
```

---

## Best Practices

### 1. Cache AI Responses
```java
// Avoid repeated API calls for same data patterns
@Cacheable("testdata")
public List<User> generateUsers(String scenario, int count) {
    return aiClient.generate(scenario, count);
}
```

### 2. Use Appropriate Models
```python
# GPT-4 for complex reasoning
complex_data = generate_with_gpt4(complex_prompt)

# GPT-3.5 for simple generation (cheaper, faster)
simple_data = generate_with_gpt35(simple_prompt)

# Local model for sensitive data
sensitive_data = generate_with_local_llm(sensitive_prompt)
```

### 3. Validate AI Output
```java
public List<User> generateValidatedUsers(int count) {
    List<User> users = aiClient.generateUsers(count);

    // Validate AI output
    return users.stream()
        .filter(this::isValidUser)
        .collect(Collectors.toList());
}

private boolean isValidUser(User user) {
    return EmailValidator.isValid(user.getEmail())
        && PhoneValidator.isValid(user.getPhone())
        && user.getName() != null && !user.getName().isBlank();
}
```

### 4. Version Your Prompts
```yaml
# prompts/user_generation_v2.yaml
version: 2
prompt: |
  Generate {count} test users for {context}.
  ...
parameters:
  temperature: 0.7
  max_tokens: 2000
```

### 5. Monitor Costs
```python
class AIDataGenerator:
    def __init__(self):
        self.token_usage = 0
        self.cost_limit = 10.0  # $10 per test run

    def generate(self, prompt):
        if self.estimate_cost(prompt) + self.current_cost > self.cost_limit:
            raise CostLimitExceeded()

        response = self.call_ai(prompt)
        self.token_usage += response.usage.total_tokens
        return response
```

---

## Resources

- [OpenAI Cookbook - Synthetic Data](https://cookbook.openai.com/)
- [Synthetic Data Vault (SDV)](https://sdv.dev/)
- [Gretel.ai Documentation](https://docs.gretel.ai/)
- [Faker Documentation](https://faker.readthedocs.io/)

---

*Part of the AI Test Strategy documentation. Contributions welcome!*
