# Create Parent project - proj

mvn archetype:generate -DgroupId=com.companyname.proj -DartifactId=proj

# Update pom.xml to declare it as a Parent project 

<packaging>pom</packaging>

# Create Sub-modules 

cd proj

## Create proj-core module.
mvn archetype:generate -DgroupId=com.companyname.proj  -DartifactId=proj-core
## Create proj-common module
mvn archetype:generate -DgroupId=com.companyname.proj  -DartifactId=proj-common
## Create proj-web module 
mvn archetype:generate -DgroupId=com.companyname.proj  -DartifactId=proj-web -DarchetypeArtifactId=maven-archetype-webapp

Now if you open the proj parent project pom.xml, you will find all three modules being added in there.

 <modules>
    <module>proj-core</module>
    <module>proj-common</module>
    <module>proj-web</module>
  </modules>

Also, in each sub-module  pom.xml, a parent section is being added.

 <parent>
    <artifactId>proj</artifactId>
    <groupId>com.companyname.proj</groupId>
    <version>1.0-SNAPSHOT</version>
  </parent>

# Update Sub-Modules pom.xml
## Open pom.xml of the proj-core module and update the packaging with jar:
<packaging>jar</packaging>

## Open pom.xml of the proj-common module and update the packaging with jar:
<packaging>jar</packaging>

## Open pom.xml of the proj-web module and update the packaging with war:
<packaging>war</packaging>

# Build Multi-Module 

mvn clean install

