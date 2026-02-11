# Maven Wrapper Guide

## What is Maven Wrapper?

Maven Wrapper (mvnw) is a shell script and batch file that provides an easy way to execute Maven commands without requiring Maven to be installed on the system. It automatically downloads and uses the correct version of Maven specified in the project configuration, ensuring consistency across different development environments.

## Key Benefits

- **No Maven Installation Required**: Developers don't need to manually install Maven on their machines
- **Version Consistency**: Ensures all team members use the same Maven version
- **CI/CD Friendly**: Simplifies continuous integration setup as the build environment is self-contained
- **Easy Onboarding**: New developers can build the project immediately after cloning

## How Maven Wrapper Works

### Components

1. **mvnw** (Unix/Linux/Mac shell script)
2. **mvnw.cmd** (Windows batch file)
3. **.mvn/wrapper/** directory containing:
   - `maven-wrapper.jar` - Small executable JAR that downloads Maven
   - `maven-wrapper.properties` - Configuration file specifying Maven version and download URL

### Execution Flow

1. Developer runs `./mvnw clean install` (or `mvnw.cmd` on Windows)
2. The wrapper script checks if the specified Maven version exists in `~/.m2/wrapper/dists/`
3. If not found, it downloads Maven from the URL specified in `maven-wrapper.properties`
4. The wrapper caches the Maven distribution locally
5. Maven executes the requested command using the downloaded/cached version

### Example maven-wrapper.properties

```properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.6/apache-maven-3.9.6-bin.zip
wrapperUrl=https://repo.maven.apache.org/maven2/org/apache/maven/wrapper/maven-wrapper/3.2.0/maven-wrapper-3.2.0.jar
```

## How to Add Maven Wrapper to a Project

### Method 1: Using Maven Wrapper Plugin (Recommended)

If you already have Maven installed, run this command in your project root:

```bash
mvn wrapper:wrapper
```

This generates all necessary wrapper files with the latest Maven version.

### Method 2: Specify Maven Version

To use a specific Maven version:

```bash
mvn wrapper:wrapper -Dmaven=3.9.6
```

### Method 3: Manual Setup

1. Create `.mvn/wrapper/` directory in project root
2. Download `maven-wrapper.jar` from Maven Central
3. Create `maven-wrapper.properties` with desired Maven version
4. Download `mvnw` and `mvnw.cmd` scripts from Apache Maven GitHub repository
5. Make `mvnw` executable: `chmod +x mvnw`

### Method 4: Copy from Another Project

You can copy the following files from an existing project with Maven Wrapper:
- `mvnw`
- `mvnw.cmd`
- `.mvn/wrapper/maven-wrapper.jar`
- `.mvn/wrapper/maven-wrapper.properties`

Then make the script executable:
```bash
chmod +x mvnw
```

## Using Maven Wrapper

### Basic Usage

Replace `mvn` with `./mvnw` (Unix/Linux/Mac) or `mvnw.cmd` (Windows):

```bash
# Clean and build
./mvnw clean install

# Run tests
./mvnw test

# Package application
./mvnw package

# Run Spring Boot application
./mvnw spring-boot:run

# Skip tests
./mvnw clean install -DskipTests
```

### Common Commands

```bash
# Compile source code
./mvnw compile

# Run specific test class
./mvnw test -Dtest=EmployeeServiceTest

# Generate project documentation
./mvnw site

# Display dependency tree
./mvnw dependency:tree

# Update dependencies
./mvnw versions:display-dependency-updates
```

## Version Control

### Files to Commit

Always commit these files to version control:
- `mvnw`
- `mvnw.cmd`
- `.mvn/wrapper/maven-wrapper.properties`

### Files to Ignore (Optional)

Some teams choose to gitignore and let developers generate locally:
- `.mvn/wrapper/maven-wrapper.jar`

However, committing the JAR ensures the project works even if Maven Central is unavailable.

### Example .gitignore Entry

```gitignore
# Maven Wrapper downloaded jar (optional - some projects commit this)
# .mvn/wrapper/maven-wrapper.jar
```

## Troubleshooting

### Permission Denied Error

If you get "Permission denied" when running `./mvnw`:

```bash
chmod +x mvnw
```

### Maven Version Change

To change the Maven version, edit `.mvn/wrapper/maven-wrapper.properties`:

```properties
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.6/apache-maven-3.9.6-bin.zip
```

Then delete the cached version and run the wrapper again:

```bash
rm -rf ~/.m2/wrapper/dists/
./mvnw clean install
```

### Network/Proxy Issues

If behind a corporate proxy, configure Maven settings in `~/.m2/settings.xml`:

```xml
<settings>
  <proxies>
    <proxy>
      <id>corporate-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.company.com</host>
      <port>8080</port>
    </proxy>
  </proxies>
</settings>
```

## Best Practices

1. **Always Use Wrapper in Scripts**: Use `./mvnw` instead of `mvn` in build scripts
2. **Document Requirements**: Update README to mention wrapper usage
3. **Commit All Files**: Include wrapper files in version control
4. **Keep Updated**: Periodically update Maven version in properties file
5. **CI/CD Configuration**: Use wrapper in CI/CD pipelines for consistency

## Current Project Status

This project already includes Maven Wrapper with the following files:
- `mvnw` - Unix/Linux/Mac executable script
- `.mvn/wrapper/` - Wrapper configuration directory

To verify the Maven version being used:

```bash
./mvnw --version
```

## Additional Resources

- [Apache Maven Wrapper Documentation](https://maven.apache.org/wrapper/)
- [Maven Wrapper GitHub Repository](https://github.com/apache/maven-wrapper)
- [Maven Official Documentation](https://maven.apache.org/)
