# How to deploy to Maven Central by Travis CI


``` console
$ git clone https://github.com/making/travis-ci-maven-deploy-skelton.git
$ cd travis-ci-maven-deploy-skelton
$ cp -r deploy <path to your project>/
$ cp .travis.yml <path to your project>/
$ cd <path to your project>

$ gem install travis
$ travis login
$ travis enable -r <username>/<repository>
<username>/<repository>: enabled :)

$ export ENCRYPTION_PASSWORD=<password to encrypt>
$ openssl aes-256-cbc -pass pass:$ENCRYPTION_PASSWORD -in ~/.gnupg/secring.gpg -out deploy/secring.gpg.enc
$ openssl aes-256-cbc -pass pass:$ENCRYPTION_PASSWORD -in ~/.gnupg/pubring.gpg -out deploy/pubring.gpg.enc

$ travis encrypt --add -r <username>/<repository> SONATYPE_USERNAME=<sonatype username>
$ travis encrypt --add -r <username>/<repository> SONATYPE_PASSWORD=<sonatype password>
$ travis encrypt --add -r <username>/<repository> ENCRYPTION_PASSWORD=<password to encrypt>
$ travis encrypt --add -r <username>/<repository> GPG_KEYNAME=<gpg keyname (ex. 1C06698F)>
$ travis encrypt --add -r <username>/<repository> GPG_PASSPHRASE=<gpg passphrase>
```

Add the following elements in your pom.xml

``` xml
    <profiles>
        <profile>
            <id>ossrh</id>
            <properties>
                <gpg.executable>gpg</gpg.executable>
                <gpg.keyname>${env.GPG_KEYNAME}</gpg.keyname>
                <gpg.passphrase>${env.GPG_PASSPHRASE}</gpg.passphrase>
                <gpg.defaultKeyring>false</gpg.defaultKeyring>
                <gpg.publicKeyring>${env.GPG_DIR}/pubring.gpg</gpg.publicKeyring>
                <gpg.secretKeyring>${env.GPG_DIR}/secring.gpg</gpg.secretKeyring>
            </properties>
            <activation>
                <property>
                    <name>performRelease</name>
                    <value>true</value>
                </property>
            </activation>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.5</version>
                        <executions>
                            <execution>
                                <id>sign-artifacts</id>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.sonatype.plugins</groupId>
                        <artifactId>nexus-staging-maven-plugin</artifactId>
                        <version>1.6.2</version>
                        <extensions>true</extensions>
                        <configuration>
                            <serverId>ossrh</serverId>
                            <nexusUrl>https://oss.sonatype.org/</nexusUrl>
                            <autoReleaseAfterClose>true</autoReleaseAfterClose>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

    <distributionManagement>
        <snapshotRepository>
            <id>ossrh</id>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
        <repository>
            <id>ossrh</id>
            <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
    </distributionManagement>
```
