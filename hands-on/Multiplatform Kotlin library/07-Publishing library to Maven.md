# Publishing library to Maven

Our first multiplatform library is almost ready. The last step is to publish it, so other projects can then depend on our library.
To make the publishing mechanism work, you should enable the experimental Gradle feature in `settings.gradle`:

<div class="language-kotlin" markdown="1" mode="groovy" theme="idea" data-highlight-only>

```groovy
enableFeaturePreview('GRADLE_METADATA')
```

</div>


Now the classic `maven-publish` Gradle [plugin](https://docs.gradle.org/current/userguide/publishing_maven.html) can be used.
Don't forget to specify the group and version of your library along with the plugin in `build.gradle`:

<div class="language-kotlin" markdown="1" mode="groovy" theme="idea" data-highlight-only>

```groovy
apply plugin: 'maven-publish'
group 'org.jetbrains.base64'
version '1.0.0'
```

</div>

Now check it with the command `./gradlew publishToMavenLocal` and you should see a successful build. 
That's it, our library is now successfully published and any Kotlin project can depend on it, whether it is another common library, JVM, JS, or Native application.

