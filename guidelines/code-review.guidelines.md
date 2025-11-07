
# Review Checklist
- **Code Quality**: Ensure the code adheres to clean code principles and coding standards.
- **Backward Compatibility**: Verify that API changes database changes, message queue changes are backward compatible.
- **Security Practices**: Ensure that security practices are followed
    - input validation, error handling
- **Tests** - Unit and Integration test written for new changes made?
- **Performance checks** - Looks for chatty api calls, chatty DB calls, indexes, caching
- **Function correctness** - based on the change itself and criticality of the code
- **Feature Toggles** - Presence of Toggles for backward compatibility
- **External Configurations**: new configs introduced or new secrets key introduced and present in secret manager.
- **Logging** - info/war/deb/error is used appropriately. Sensitive data not present in logs
- **Integration Points** Have connection timeout, read timeout, write timeout be set accordingly

# Details:

## Clean Code Practices

Clean code is foundational for readability and maintainability, reducing the cognitive load on developers. Key practices include:

- **Meaningful Names**: Variables, functions, and classes should have descriptive names that reveal intent. For example, use calculateTotalPrice instead of calc.
    - Ref - [Clean Code in Java](https://www.baeldung.com/java-clean-code)
- **Small Functions**: Functions should be short, ideally under 20 lines, and focus on a single responsibility.
    - Ref - [How to Write Clean Code](https://www.freecodecamp.org/news/how-to-write-clean-code/)
- **Modularization**: Break down code into smaller, reusable modules.
    - Ref - [Java Clean Code](https://www.scaler.com/topics/java/java-clean-code/)
- **Documentation**: Use comments for complex logic and maintain Javadoc where necessary
- **Consistent Formatting**: Adhere to standard formatting using tools like Checkstyle.
- **Avoiding High Cyclomatic Complexity**: Keep functions simple with fewer conditional paths to reduce complexity. Cyclomatic complexity measures the number of linearly independent paths through code, and high values (e.g., >10) indicate potential maintenance issues. Refactor complex conditionals into smaller functions.
- **Tell, Don't Ask Principle**: Objects should handle their own logic rather than exposing data for others to manipulate. For example, instead of if (order.getStatus().equals("pending")) order.setStatus("processed"), use order.process().
    - Ref - [Clean Code Best Practices](https://medium.com/@samuelcatalano/clean-code-best-practices-and-examples-in-java-e04e1ae70835).

**Example Table: Clean Code Checklist for Reviewers**

**Check** | **Description** |
| --- | --- |
| Meaningful Names | Are names descriptive and follow conventions? |
| Function Size | Is function under 50 lines and focused? |
| Modularization | Are modules loosely coupled? |
| Documentation | Are complex parts commented with Javadoc? |
| Formatting | Is indentation consistent? |
| *****Cyclomatic Complexity*** | Is function complexity low? |
| Tell, Don't Ask Principle | Does the code tell objects what to do? |

## Database Changes

For PostgreSQL, managing schema and data changes is critical.

Reviewers should ensure:

- ****Schema changes follow backward compatibility principles***
    - New columns are nullable with appropriate default values
    - Column type changes include data migration plan
- Tables/columns are not removed without proper deprecation strategy
- Database migrations are idempotent and can be run multiple times safely
- Rollback scripts are provided for each migration
- ****Indexes are added for columns used in WHERE clauses or JOINs***
- Foreign key constraints are properly defined with appropriate ON DELETE/UPDATE actions

Ensure backward compatibility*:

## API Changes

Checklist:

- **Check for API Style guides**
- Service boundaries are respected (no direct database access to other services)
- **Check for backward compatibility, is versioning needed**
- Check for business logic change behind toggles
- Check for logging aspects, distributed traces present.
- Check for http status code responses
- Authentication and authorization checks are implemented

## Message Queue Changes:

Ensure backward compatibility is maintained 

## Security:

Checklist:

- **Sensitive information (passwords, accountNos, PII) not logged**
- Sensitive data protected at rest and in transit with encryption
- **Secrets scanning in local before remote push - Talisman**
- Auth/Authorization implemented
- Sanitize inputs to prevent SQL injection and XSS
- Is Secret rotation for database implemented?

## Testing

Though not explicitly requested, testing is crucial for clean code and quality assurance. Reviewers should ensure:

- **Unit Testing**: Test components in isolation using JUnit, as per [JUnit Documentation](https://junit.org/). Mock dependencies with Mockito for isolation.
- **Integration Testing**: Test service interactions, ensuring system-wide functionality.