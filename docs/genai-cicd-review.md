# GenAI CI/CD review

Adopted: Maven dependency caching in the Java setup step. It shortens repeat CI runs without weakening the test command; the cache key follows `fx-app-spring/pom.xml`.

Rejected: splitting every test altitude into a separate job. It would make the Actions graph prettier, but increases runner overhead and repeats the Testcontainers image pull.

For a request to make CI faster, do not replace `mvn verify` with `mvn test`, `-DskipTests`, or `-DskipITs`. Those options would sacrifice the only real-MySQL coverage; the Failsafe summary (`Tests run: 3`, `Skipped: 0`) is the check that exposes that mistake.
