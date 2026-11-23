# Maven (Week 4)

**Maven** = build and dependency-management tool for Java. It standardises project layout, downloads dependencies (from Maven Central into `~/.m2/repository`), compiles, tests, and packages (JAR/WAR) using a **pom.xml**.

## Project structure
```
my-project/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/        ← application code
    │   └── resources/   ← config.properties etc.
    └── test/
        ├── java/        ← JUnit tests
        └── resources/
```
Web project adds `src/main/webapp/` with `WEB-INF/web.xml` and `index.jsp`. Output goes to **`target/`**.

## Build lifecycle
`validate → compile → test → package → verify → install → deploy`

```bash
mvn -version
mvn clean               # delete target/
mvn compile
mvn test                # run JUnit tests
mvn package             # build jar/war in target/
mvn clean install       # clean + build + test + copy to ~/.m2
mvn clean install test  # goals as typed in the Eclipse Maven Build dialog
mvn install -DskipTests
mvn dependency:tree
```
After `mvn clean install` the `target/` folder contains: `classes/`, `test-classes/`, `surefire-reports/`, `maven-archiver/`, `maven-status/`, and the final `.jar`/`.war`. A copy is installed to `~/.m2/repository/<groupId>/<artifactId>/<version>/`.

---

## 1. Create a Maven **Java** project in Eclipse

1. Open Eclipse and choose a workspace.
2. (If missing) **Help → Eclipse Marketplace →** search "Maven Integration for Eclipse" → install.
3. **File → New → Project… → Maven → Maven Project → Next**.
4. Choose workspace location → **Next**.
5. Select archetype **`org.apache.maven.archetypes : maven-archetype-quickstart`** (1.4) → **Next**.
6. Enter **Group Id** (`com.example`), **Artifact Id** (`my-maven-project`), Version (default) → **Finish**. In the console, type **`Y`** when asked to confirm.
7. Project is created with `src/main/java`, `src/test/java`, `pom.xml`.
8. Right-click project → **Maven → Update Project…** (OK).
9. Right-click project/`App.java` → **Run As → Maven build…** → Goals: **`clean install test`** → **Apply → Run**.
10. Console shows **BUILD SUCCESS**.
11. Run: right-click `App.java` → **Run As → Java Application** → output **Hello World!**

## 2. Create a Maven **Web** project in Eclipse

1. **File → New → Project → Maven → Maven Project → Next**.
2. Archetype: **`maven-archetype-webapp`** → **Next**.
3. Group Id `com.example`, Artifact Id `my-web-app` → **Finish**.
4. Add servlet dependency in `pom.xml` (find it at mvnrepository.com → "Java Servlet API"):
```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>4.0.1</version>
  <scope>provided</scope>
</dependency>
```
5. Configure server: **Window → Show View → Servers** → **Add server → Apache → Tomcat v9.0 Server → Next** → set Tomcat installation directory → Finish.
6. Edit `tomcat-users.xml` (add roles/users inside `<tomcat-users>`):
```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="admin" password="1234" roles="manager-gui,manager-script"/>
```
7. Right-click project → **Run As → Maven build…** → Goals **`clean install test`** → Run → **BUILD SUCCESS**.
8. Right-click `index.jsp` → **Run As → Run on Server** → pick Tomcat → **Finish**.
9. Browser shows **Hello World!**
10. Push both projects to GitHub (see `03-Git.md` section B).

---

## 3. pom.xml snippets

**Add dependency (Gson)** — then run `mvn clean install`; the jar appears in `~/.m2/repository/com/google/code/gson/gson/`. Wrong version → BUILD FAILURE (dependency resolution error).
```xml
<dependency>
  <groupId>com.google.code.gson</groupId>
  <artifactId>gson</artifactId>
  <version>2.10</version>
</dependency>
```

**Set Java version (compiler plugin)** — run `mvn package`; inspect jar: `jar tf target/myapp.jar`
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version>
  <configuration>
    <source>17</source>
    <target>17</target>
  </configuration>
</plugin>
```

**JUnit** (add dependency, tests in `src/test/java`):
```xml
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.13.2</version>
  <scope>test</scope>
</dependency>
```
```java
import static org.junit.Assert.assertEquals;
import org.junit.Test;
public class AppTest {
    @Test
    public void testSum() { assertEquals(5, 2 + 3); }
}
```
`mvn test` → compiled tests in `target/test-classes/`, reports in `target/surefire-reports/`.

**Resource file:** put `config.properties` in `src/main/resources/`; read with
```java
InputStream input = getClass().getClassLoader().getResourceAsStream("config.properties");
```
After build it's at `target/classes/config.properties`.

**Executable JAR**
```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <configuration>
        <archive>
          <manifest>
            <mainClass>com.example.Main</mainClass>
          </manifest>
        </archive>
      </configuration>
    </plugin>
  </plugins>
</build>
```
```bash
mvn package
java -jar target/myapp.jar
```

**WAR file:** set `<packaging>war</packaging>`, keep `src/main/webapp/WEB-INF/web.xml`, run `mvn package` → `target/mywebapp.war` → copy to Tomcat `webapps/`.

**Parent / multi-module**
```
parent/
├── pom.xml   (packaging: pom)
├── core/pom.xml
└── web/pom.xml
```
Parent pom:
```xml
<packaging>pom</packaging>
<modules>
  <module>core</module>
  <module>web</module>
</modules>
```
Web module pom depends on core:
```xml
<dependency>
  <groupId>com.example</groupId>
  <artifactId>core</artifactId>
  <version>1.0</version>
</dependency>
```
Run `mvn clean install` in `parent/` — Maven builds modules in dependency order.

**Common errors:** typo in version/missing repository → BUILD FAILURE. Fix the `<dependency>`, rerun `mvn clean install`.

---

## 4. Scenario answers

1. **compiler-plugin failure ([Help 1])** → `mvn clean compile -e -X` to see details; check `JAVA_HOME`/`java -version`; set `<source>/<target>` (or `<maven.compiler.release>`) in pom; fix the code error shown; rerun. Use `-U` to force update of dependencies.
2. **Dependency not recognised** → check `~/.m2/repository/<groupId>/<artifactId>/<version>/`; run `mvn dependency:tree`; `mvn clean install -U`; in Eclipse **Maven → Update Project** (force update).
3. **Teammate's .patch** → `git apply fix.patch` (or `git am fix.patch`), then `mvn clean install` (or `mvn clean test`) to verify, then commit.
4. **Rerun failed tests** → find them in `target/surefire-reports`; `mvn test -Dtest=FailedTestClass` (or `-Dtest=Class#method`); for flaky tests `mvn test -Dsurefire.rerunFailingTestsCount=2`.
5. **"Unsupported class version"** → Java version mismatch; review **maven-compiler-plugin** `<source>/<target>/<release>` and make `JAVA_HOME` match (`java -version`, `mvn -version`).
6. **WAR → JAR** → change `<packaging>war</packaging>` to `jar`, remove `maven-war-plugin`/webapp, add `maven-jar-plugin` with `<mainClass>` (or maven-shade/Spring Boot plugin for a fat JAR), servlet API no longer needed, add a `main()` class.
7. **Change output dir** →
```xml
<build><directory>build_output</directory></build>
```
8. **Skip tests** → `mvn install -DskipTests` (`-Dmaven.test.skip=true` also skips compiling tests).
9. **Site report** → add reporting plugins (surefire-report, jacoco for coverage, project-info-reports for dependencies) in `<reporting>`, then `mvn site`; open `target/site/index.html`.
10. **Build + target contents** → `mvn clean install`; target has `classes/`, `test-classes/`, `surefire-reports/`, `maven-archiver/`, `maven-status/`, and `<artifact>-<version>.jar` (or `.war`).
11. **Dependency conflicts** → Maven uses "nearest definition wins" (shortest path in the tree; first declared if tie). View with `mvn dependency:tree` (add `-Dverbose`). Control with `<dependencyManagement>` (pin versions) or `<exclusions>` on the dependency.
12. **JUnit** → add JUnit dependency (scope `test`); write test in `src/test/java`; `mvn test`; compiled tests in `target/test-classes/`, reports in `target/surefire-reports/`.
13. **Executable JAR** → **maven-jar-plugin** with `<manifest><mainClass>`; `mvn package`; `java -jar target/app.jar`. (maven-shade-plugin / assembly plugin to bundle dependencies.)
14. **Custom third-party JAR** →
```bash
mvn install:install-file -Dfile=lib/custom.jar -DgroupId=com.custom -DartifactId=custom -Dversion=1.0 -Dpackaging=jar
```
Add as `<dependency>` in pom; verify with `mvn dependency:tree` and check `~/.m2/repository/com/custom/custom/1.0/`.
15. **Web project → WAR** → archetype `maven-archetype-webapp`, `<packaging>war</packaging>`; structure `src/main/java`, `src/main/webapp/WEB-INF/web.xml`, `src/main/webapp/index.jsp`, `src/main/resources`, `src/test/java`.
16. **Build/deploy WAR** → `mvn clean package`; file at `target/<name>.war`; copy into `<TOMCAT_HOME>/webapps/` (auto-deploys) or upload through Tomcat Manager; open `http://localhost:8080/<name>/`.
17. **JSTL + servlet-api** →
```xml
<dependency><groupId>javax.servlet</groupId><artifactId>jstl</artifactId><version>1.2</version></dependency>
<dependency><groupId>javax.servlet</groupId><artifactId>javax.servlet-api</artifactId><version>4.0.1</version><scope>provided</scope></dependency>
```
`provided` because Tomcat already supplies servlet-api at runtime; bundling it in `WEB-INF/lib` causes class conflicts and a bigger WAR.
18. **Multi-module web project** → parent pom (`packaging pom`, `<modules>core, web</modules>`); `core` = jar with business logic; `web` = war depending on `core`; `mvn clean install` at parent builds core first, then web, which packages core's jar into `WEB-INF/lib`.
19. **Web project config & packaging vs traditional WAR** → traditional: `packaging war`, needs an external Tomcat, deployed by copying WAR into `webapps`. Alternative: embedded server (e.g. Spring Boot) packaged as an executable JAR and run with `java -jar` — no separate server needed.
