---
published: false
---
# Spring Boot - testing Spring Data repositories with Spock


In this article, I would like to present you with a simple, yet very effective approach for testing your Spring Data repositories using the Spock framework. For the purposes of this article, we will be using the Spring Data Neo4j module to create our simple DAO and repository and Gradle for handling our dependencies. 

## Why Spock?

Spock is a very useful framework for testing your application - it has a very simple, Groovy-based syntax, providing its user with much more flexibility when it comes down to creating test cases and checking its results.

# Project setup

## Create your Spring Boot project

In order to create your project, head over to the [Spring Initializer](https://start.spring.io/) website. You will be asked to select adequate options for your project - just as shown on the image below.
For now, we will ignore the dependencies section. Feel free to change the following options such as the Java version according to your needs.

![spring_init.png]({{site.baseurl}}/_posts/spring_init.png)

Next, press the Generate button - this will initiate a download of a .zip archive containg all of the necessary files for this sample project. Unpack the archive in a location of your choice. 

## Add dependencies

Once you're done unpacking the archive, head over to the root directory of your project and open the `build-gradle` file. Here, we will want to add some dependencies for our project. 
Inside the `dependencies` section add the following:
- `implementation 'org.springframework.boot:spring-boot-starter-data-neo4j'` - this will allow us to create models of Neo4j nodes and repositories.

## Estabilish a connection with your database

Head over to `src/main/resources` and open the `application.properties` file. Here, you will need to define your database's address and credentials for estabilishing a connection. You will need access to a running database for this step.

The file should look somewhat like this:
```
spring.neo4j.uri=<your bolt address>
spring.neo4j.authentication.username=<your username>
spring.neo4j.authentication.password=<your password>
```

## Create a simple Data Access Object (DAO)

Before we create a repository, wee need a model of the repository's contents - in this example, we will create a simple Person DAO. Inside the `src/main/java` folder create a new file `Word.java`:

Make sure to create:
- a getter and a setter for the `word` field
- `hashCode()` and `equals()` methods for future comparisons of the objects

Note that there is no need to create any constructors.

Your Word.java file should look somewhat like this:

```
import org.springframework.data.neo4j.core.schema.Id;
import org.springframework.data.neo4j.core.schema.Node;
import java.util.Objects;

@Node
public class Word
{
    @Id
    private String word;

    public String getWord()
    {
        return word;
    }

    public void setWord(String word)
    {
        this.word = word;
    }

    @Override
    public boolean equals(Object o)
    {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Word word1 = (Word) o;
        return Objects.equals(getWord(), word1.getWord());
    }

    @Override
    public int hashCode()
    {
        return Objects.hash(getWord());
    }
}

```

## Create a simple Word repository:
Now, we will create a simple repository model for our Word entity - it will 


# Setup Spock

In order to use the Spock framework, we have to edit the `build-gradle` file once more to add missing dependencies. Inside the `dependencies` section add the following:
- `testImplementation 'org.spockframework:spock-core:2.0-groovy-3.0'`
- `testImplementation 'org.spockframework:spock-spring:2.0-groovy-3.0'`




