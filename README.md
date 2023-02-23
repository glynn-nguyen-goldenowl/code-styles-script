# code-styles-script
# ktlint
## Setup steps:
### 1. Add lines into the root level build.gradle
```
plugins {
    id "org.jlleitschuh.gradle.ktlint" version "11.2.0"
    id("org.jlleitschuh.gradle.ktlint-idea") version "11.2.0"
}
```
### 2. Add lines into app level build.gradle
```
apply plugin: 'org.jlleitschuh.gradle.ktlint'

// copy pre-commit file into git hooks folder.
task installGitHook(type: Copy){
    from new File(rootProject.rootDir , "pre-commit")
    into {new File(rootProject.rootDir, ".git/hooks")}
    fileMode 0775
}
// check the code style before commit, if check style failed, the commit will be canceled.
tasks.getByPath('preBuild').dependsOn installGitHook
// auto format code before build apk
tasks.getByPath("preBuild").dependsOn("ktlintFormat")

ktlint{
    android = true
    ignoreFailures = false
    disabledRules = ["no-wildcard-imports"]
    // to disableRule for unittest with max-length
    additionalEditorconfigFile = rootProject.file("./editorConfig")
    reporters {
        reporter "plain"
        reporter "checkstyle"
        reporter "sarif"
    }
}
```

### 3. In the project root folder, create editorConfig file
```
[*.{kt,kts}]
ktlint_code_style = ktlint_official
ktlint_ignore_back_ticked_identifier = true
```

### 4. In the project root folder, create pre-commit
```
#!/bin/bash

echo "Running lint check..."

./gradlew app:ktlintCheck --daemon

status=$?

# return 1 exit code if running checks fails
[ $status -ne 0 ] && exit 1
exit 0
```
